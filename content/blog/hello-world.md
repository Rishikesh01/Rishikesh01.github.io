---
title: "Hello, world"
date: 2026-08-09
description: "Why this blog exists, and what to expect here."
---

I spend my days making Go services faster and harder to kill — and some of the most
useful things I know, I learned from problems that took days to root-cause and fit
into a two-line summary afterward. Those summaries deserve a home. This is it.

Expect posts about:

- **Go in production** — concurrency bugs, job queues, and the sharp edges of `database/sql`
- **Reliability** — what actually breaks (spoiler: it's the failover), and what fixed it
- **Performance** — profiling stories with before/after numbers, because numbers keep me honest

Here's the obligatory first snippet, in the language this blog will mostly be about:

```go
func main() {
	fmt.Println("hello, world")
}
```

That's it for now. The first real post will dig into how a Postgres failover under
asynchronous replication quietly lost us acknowledged writes — and why synchronous
commit was the right trade.
