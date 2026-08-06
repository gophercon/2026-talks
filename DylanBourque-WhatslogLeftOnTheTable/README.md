# What `slog` Left On The Table - Dylan Bourque, CoreWeave

## Talk Description

In 2023, Go 1.21 officially introduced the `log/slog` package, real structured logging in the standard library that many of us had been waiting years for. It covers the fundamentals well, but there are gaps that can become painful in real-world, large-scale systems. This talk explores two extensions, built on the `slog.Handler` interface, that fill some of those gaps. Some Gophers might even argue they should be part of the `log/slog` package itself.

First, a custom `slog.Handler` implementation that integrates `slog` with `testing.T`, routing the logs to the test's output. This has the nice effect of suppressing log noise during passing tests while surfacing full "production" logs on failure. We will cover the handler code itself, then dive into several tricky edge cases and how they were addressed.

Second, a dynamic per-component log leveling system that lets you change the log verbosity for individual components or subsystems at runtime, with no restarts and no performance penalty. If you've ever enabled DEBUG logging for your service to troubleshoot an issue and been buried in an avalanche of logs completely obscuring the ones you care about, you won't want to miss this.

## Speaker Info

Dylan has been a professional software developer since 1998, working in the banking, health care, finance, computer security, developer tooling, and now AI industries.  He’s been writing Go full time since early 2016. He is a part-time co-host of the Fallthrough podcast and is an active participant in the Gophers Slack organization. This will be his ninth GopherCon and his fifth time on stage.  Outside of work, Dylan enjoys baseball and golf.

## Supporting Materials

- `slogext` source on GitHub: [dylan-bourque/slogext](https://github.com/dylan-bourque/slogext)
- `slog` Handler Guide: [the Go wiki](https:golang.org/s/slog-handler-guide)
- `TestHandler` language proposal: [golang/go/issues/80138](https://github.com/golang/go/issues/80138)
- Presentation slide deck: [here](./slide_deck.pptx).
