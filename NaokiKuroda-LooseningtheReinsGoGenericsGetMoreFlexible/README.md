# Loosening the Reins: Go Generics Get More Flexible - Naoki Kuroda, IBM

## Talk Description

While reading through the Go 1.26 release notes, one declaration caught my attention: `type Ordered[T Ordered[T]]`, a type whose constraint refers back to itself. Go 1.26 accepts it, but Go 1.25 rejected it as an invalid recursive type. I wanted to know why the rule changed, so I dug through the specification history, issue reports, code reviews, and the `go/types` checker to find out why the restriction was there in the first place, and why it was finally safe to remove.

That one example turned out to be part of a bigger pattern. I also walk through how the generics rules were relaxed in Go 1.20, Go 1.25, and the Go 1.27 preview, asking the same question each time: what problem was the original rule actually guarding against? The answer, again and again, is that a restriction should be no broader than the problem it prevents. If you have ever run into a puzzling "this is not allowed" from the type checker, you will come away with a better sense of where those rules come from and why they keep loosening.

## Speaker Info

Naoki is a backend engineer at IBM in Japan and an active member of the Go community.

## Supporting Materials

- Slides:
  - [Speaker Deck](https://speakerdeck.com/kuro_kurorrr/loosening-the-reins-go-generics-get-more-flexible)
  - [Download as a PDF](./slides.pdf)
