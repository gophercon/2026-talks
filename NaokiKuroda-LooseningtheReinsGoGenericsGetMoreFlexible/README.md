# Loosening the Reins: Go Generics Get More Flexible - Naoki Kuroda

## Talk Description

While reading through the Go 1.26 changes, one small declaration stood out: a type whose constraint refers back to itself, `type Ordered[T Ordered[T]]`. Go 1.26 accepts it, but every release through Go 1.25 rejected it as an invalid recursive type. This talk follows the trail behind that one-line difference, through the specification history, issue reports, code reviews, and the `go/types` checker, to explain why the restriction existed, why it was finally safe to lift, and how the compiler recognizes the cycle today.

From there we widen the lens to three more refinements of the generics rules: how Go 1.20 separated constraint satisfaction from implementation so `any` can be used where `comparable` is required, how Go 1.25 replaced core types with per-operation rules, and how the Go 1.27 preview lets concrete methods declare their own type parameters. Each change tells the same story from a different angle, that a restriction should be no broader than the problem it prevents. You will leave with a sharper mental model of how Go's generics actually work, and why they keep getting more flexible.

## Speaker Info

Naoki Kuroda is a backend engineer from Japan and an active member of the Go community.

## Supporting Materials

- Slides:
  - [View online](https://speakerdeck.com/kuro_kurorrr/loosening-the-reins-go-generics-get-more-flexible)
  - [Download as a PDF](./slides.pdf)
