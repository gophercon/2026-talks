# Unlocking Real-World Go Mutex Usage Patterns by Writing a Mutex Linter - Vladimir Dementyev, Evil Martians

## Talk Description

The trickiest and hardest-to-identify bugs are often those fixable with a single line of code. Hours, days, or months can be spent only to find a missing `Unlock()` in the (dead) end. After I suffered such a case, I asked myself two questions: "Am I using mutexes wrong?" and "How can I prevent bugs like this in the future?"

The search for answers led me to the depths of static analysis and an exploration of Go mutex usage patterns in the wild.

In my talk, I want to share my findings and present a new linter for detecting mutex misuse.

## Speaker Info

Vladimir is a mathematician who found his happiness in programming Ruby and Go, contributing to open source, and being an Evil Martian.

## Supporting Materials

- Mulint: [GitHub](https://github.com/palkan/mulint).
- Presentation slide deck: [here](./slides.pdf).
