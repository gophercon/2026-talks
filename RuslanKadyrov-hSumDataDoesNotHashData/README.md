# h.Sum(data) Does Not Hash Data - Ruslan Kadyrov, Mercari, Inc.

## Talk Description

In this lightning talk, I'll walk through how I misunderstood Go's
`hash.Hash` interface, why the mistake looked perfectly reasonable, and what
it reveals about Go's API design. We'll explore common hashing footguns, the
difference between `sha256.Sum256` and streaming hashers, and why small
misunderstandings in low-level APIs can quietly produce incorrect behavior in
production systems.

**Talk/Attendee Level:** All Attendees

## Speaker Info

Ruslan Kadyrov is a software engineer focused on backend systems and
infrastructure, building large-scale distributed systems with Golang, Kubernetes,
and gRPC at Mercari, Inc., Tokyo, Japan. He spends most of his time dealing
with the less glamorous parts of "clean" abstractions: production outages,
security boundaries, and interfaces that looked great in code review but
failed under real load. He enjoys turning painful production incidents into
concrete engineering lessons and sharing what actually worked and what didn't.

## Supporting Materials

- [Slides (PDF)](gc2026_h-sum-does-not-hash-data.pdf)
- [Session page](https://www.gophercon.com/agenda/session/1801590)
- Blog: [r-kadyrov.hashnode.dev](https://r-kadyrov.hashnode.dev/)
