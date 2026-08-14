# Nip the Nits in the Bud - Garrett Denis, Skool

## Abstract

Repeated "nit" comments (line wrapping, formatting, style preferences)
are easy to follow but tedious to enforce by hand, and every one of them
steals reviewer attention from correctness, design, and risk. In this
lightning talk I'll turn one real nit from our codebase, enforcing one
clause per line in boolean conditionals, into a custom Go linter using
`go/analysis`, complete with an auto-fix and golangci-lint integration,
in a single worked example. The point isn't the rule, it's that the
pattern is repeatable. If a style choice is objective and recurring, it
belongs in tooling, not a review thread. As AI assistants generate ever
more code, codifying your team's conventions as lint rules is how you
keep both humans and machines consistent, and keep review focused on
what actually matters.

## Speaker Info

Garrett is a Software Engineer at Skool where he works on Go backend
services that help people build and find their online community.
Previously, he worked on platform teams at IDme, CrowdStrike, and AWS
delivering highly-available services at scale. He’s especially
interested in developer experience and using tools like CI, linting, and
automation to shorten feedback loops and keep code review focused on
correctness and maintainability instead of style debates.

Outside of work, Garrett is based in Southern California and is usually
looking for an excuse to get outside and away from the screen, whether
that’s running, hiking, backpacking, or anything in between.

## Supporting Materials

- [Slides (PDF)](slides.pdf)

## The nit

The review comment the talk starts from:

```go
// flagged: four clauses on one line
if u != nil && u.Active && u.Verified && (u.Role == "admin" || ownerID == postID) {

// what the rule asks for
if u != nil &&
	u.Active &&
	u.Verified &&
	(u.Role == "admin" ||
		ownerID == postID) {
```

Each clause gets its own line, so `git blame` points at one idea and a
review comment can point at exactly one of them.

The part that made this worth automating: the convention was settled at
Skool years before I joined, and the pull requests I kept seeing it on
were not from new hires. Long-tenured engineers were on both sides of
the same comment, forgetting it one week and catching it the next. That
is not a knowledge gap and not a disagreement, it is decay, and you
cannot fix decay by saying the thing again.

## The whole analyzer

`go/analysis` gives you an `Analyzer` value and a `run` function. Off
the `*analysis.Pass`, a syntax rule like this one needs very little:
`pass.Files` for the parsed ASTs, `pass.Fset` to turn a position into a
line, and a way to report.

```go
const msgOperandsSharedLine = "operands of && or || must each start" +
	" on a different line"

var Analyzer = &analysis.Analyzer{
	Name: "logicalchain",
	Doc: "enforces one clause per line for && / || binary" +
		" expressions",
	Run: run,
}
```

The rule itself is two positions and a comparison. `a && b` parses to a
`*ast.BinaryExpr`, so for every one joined by `&&` or `||`, ask where
the left side ends and where the right side starts:

```go
b, ok := n.(*ast.BinaryExpr)
if !ok {
	return true
}
// only the logical operators; bitwise & and | are a different
// token and must not be touched
if b.Op != token.LAND &&
	b.Op != token.LOR {
	return true
}

lhsEnd := pass.Fset.Position(b.X.End()).Line
rhsStart := pass.Fset.Position(b.Y.Pos()).Line
if lhsEnd == rhsStart {
	// the clauses share a line: report it
}
```

No string parsing, no regex over source, no counting columns.

## The auto-fix

A `SuggestedFix` is a list of `TextEdit`s, and a `TextEdit` is a byte
range plus the bytes to put there. For this rule the replacement is a
newline and `gofmt` handles the indentation afterwards:

```go
// splitAfter returns the offset just past b's operator, which is
// where a newline has to go to move b's right operand onto its own
// line.
func splitAfter(b *ast.BinaryExpr) token.Pos {
	return b.OpPos + token.Pos(len(b.Op.String()))
}

edits := make([]analysis.TextEdit, 0, len(ops))
for _, b := range ops {
	at := splitAfter(b)
	edits = append(
		edits,
		analysis.TextEdit{
			Pos:     at,
			End:     at, // an insertion, not a replacement
			NewText: []byte("\n"),
		},
	)
}

pass.Report(
	analysis.Diagnostic{
		Pos:     first.OpPos,
		Message: msgOperandsSharedLine,
		SuggestedFixes: []analysis.SuggestedFix{
			{
				Message:   "put each operand on its own line",
				TextEdits: edits,
			},
		},
	},
)
```

Two details in there were bugs first, and both are worth stealing:

**A chain nests, so one line can need several edits.** `a || b || c`
parses as `(a || b) || c`, and `ast.Inspect` is preorder, so the *outer*
operator is visited first. My first version deduplicated diagnostics by
line, which meant only that outer operator got an edit. Running `--fix`
on `if v0 || v1 || v2 {` produced this, splitting one clause and leaving
the other two joined:

```go
if v0 || v1 ||
	v2 {
```

The fix is to collect every offending operator on a line and attach an
edit for each, while still reporting one diagnostic per line so the
message does not repeat three times.

**`End == Pos` makes it an insertion.** The obvious version replaces the
span between the operator and the next clause:

```go
Pos: splitAfter(b),
End: b.Y.Pos(),  // don't do this
```

That deletes whatever is sitting in the span, so
`if a || /* keep me */ c {` silently loses its comment. An empty range
inserts instead and touches nothing.

## Testing it

`analysistest` compiles fixture packages, runs the analyzer, and diffs
the diagnostics against `// want` comments. Two packages: `valid` must
produce nothing, `invalid` is tagged.

```go
func TestAnalyzer(t *testing.T) {
	analysistest.Run(
		t,
		analysistest.TestData(),
		logicalchain.Analyzer,
		"valid",
		"invalid",
	)
}

// applies the fixes and diffs the result against
// testdata/src/invalid/invalid.go.golden
func TestSuggestedFixes(t *testing.T) {
	analysistest.RunWithSuggestedFixes(
		t,
		analysistest.TestData(),
		logicalchain.Analyzer,
		"invalid",
	)
}
```

The `valid` package earns its keep more than the `invalid` one, because
false positives are what get a linter disabled. Mine covers bitwise
`&`/`|` on a shared line, already-split chains, and nested parens:

```go
// must NOT be flagged, however many share a line
a, bb, c := 1, 2, 3
_ = a & bb & c
_ = a | bb | c
_ = a&bb | c
```

One habit worth stealing: once the tests pass, break the analyzer on
purpose and confirm the fixtures fail. Both of the bugs above pass a
naive test suite. The golden test only caught the comment-swallowing one
after I added a fixture with a comment in that exact position.

## Shipping it

Nobody runs a one-off binary, so it has to live in the linter the team
already runs. golangci-lint module plugins are three small pieces.

A `.custom-gcl.yml` naming the golangci-lint version to build against,
the binary you want out, and your module:

```yaml
version: v2.11.4
name: mylint
plugins:
  - module: 'example.com/mylint'
    path: ./
```

A `register.Plugin` call so golangci-lint can find the analyzer. This
rule is pure syntax, so it asks for the cheaper load mode:

```go
func init() {
	register.Plugin("logicalchain", newPlugin)
}

func newPlugin(any) (register.LinterPlugin, error) {
	return plugin{}, nil
}

type plugin struct{}

func (plugin) BuildAnalyzers() ([]*analysis.Analyzer, error) {
	return []*analysis.Analyzer{Analyzer}, nil
}

func (plugin) GetLoadMode() string {
	return register.LoadModeSyntax
}
```

Then `golangci-lint custom` builds a binary named `mylint` with your rule
compiled in, and consumers enable it in `.golangci.yml` like any other
linter:

```yaml
linters:
  enable:
    - logicalchain
  settings:
    custom:
      logicalchain:
        type: module
        original-url: example.com/mylint
```

Because it registers under its own linter name,
`//nolint:logicalchain` suppresses exactly this rule and nothing else,
which is the difference between a check people keep and a check people
turn off.

## References

- [`golang.org/x/tools/go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis)
  and the [`analysistest`](https://pkg.go.dev/golang.org/x/tools/go/analysis/analysistest)
  harness
- [golangci-lint module plugins](https://golangci-lint.run/plugins/module-plugins/)
- [`singlechecker`](https://pkg.go.dev/golang.org/x/tools/go/analysis/singlechecker)
  if you just want to run one analyzer against a package while you are
  building it
