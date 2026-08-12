# Sandboxing DuckDB with io/fs - Achille Roussel, Firetiger

## Talk Description

Go programs that embed C libraries face a hidden trap: the C library brings its own I/O stack. It doesn't know about your credentials, your TLS configuration, your request tracing, or your retry logic. It just reaches out to the network or disk on its own terms. You end up with two parallel I/O systems in the same binary, and they will drift apart in ways that are hard to debug and expensive to maintain.

DuckDB is a great example. It is one of the most useful embedded databases available today, and many Go programs are starting to use it. But DuckDB ships extensions like httpfs and aws that perform network I/O directly from C++. If your Go program wraps those extensions, you have lost observability, lost credential management, and lost the ability to apply standard Go middleware to the database's I/O operations.

The solution is simpler than it looks. Go's io/fs interface, introduced in 1.16, gives us a clean, composable abstraction for read-only file systems. Most C libraries that do I/O support some form of virtual file system hook. If you can connect those two things, you can route all of the library's I/O back through the Go runtime, and suddenly every cloud SDK, every telemetry wrapper, every retry policy you have already written becomes available to the database too.

This talk teaches you how to do that. We will walk through the split stack problem in concrete terms, look at how DuckDB's virtual file system API works, and build a mental model of the bridge: a small C++ extension that intercepts DuckDB's file operations and forwards them to a Go function that satisfies io/fs. By the end, you will understand the pattern well enough to apply it to other C libraries, and you will learn about an open-source library that does it for DuckDB today.

## Speaker Info

Achille is the Co-Founder and CTO of [Firetiger](https://firetiger.com), where they develop AI agents that verify code changes in production. He has been writing Go since 2014 and is an open-source author and contributor to many projects, including the Go runtime and compiler.

## Supporting Materials

- `go-duckfs` source on GitHub: [firetiger-oss/go-duckfs](https://github.com/firetiger-oss/go-duckfs)
- Presentation on Google Slides: [Sandboxing DuckDB with io/fs](https://docs.google.com/presentation/d/1W6oxa1l37BnPzv26bqRJ7MTuHJBhsgLJjIo6Z6kFnok/edit?usp=sharing)
- Presentation slide deck: [here](./slide_deck.pptx)
