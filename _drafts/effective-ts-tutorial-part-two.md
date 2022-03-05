---
layout: post
title: Effective.ts Tutorial - Part Two
author: David Timms
categories: typescript effective.ts
license: null
---

Welcome to the second part of our tour of [effective.ts](https://github.com/DavidTimms/effective.ts), a library for writing safe, concurrent, fault-tolerant programs in TypeScript.

In [part one](/effective.ts/2021/12/23/effective-ts-tutorial-part-one.html), we saw how we can represent actions using the `IO` type, combine those actions into full programs, and handle errors with type safety. This time, we'll see how structuring our programs in this way allows more control over concurrency, cancellation and fault-tolerance. 

## Concurrency

The methods [we looked at last time](/typescript/effective.ts/2021/12/23/effective-ts-tutorial-part-one.html#combining-actions), such as `andThen`, let you combine actions so each one happens after the previous one has finished. That is useful, but to write efficient programs, we sometimes need to perform two or more actions at the same time. The main tool which effective.ts provides to achieve this is called a `Fiber`. This is a lot like a operating system thread, except much more lightweight.

What are fibers? How to start a fiber. How to wait for a fiber's outcome.

Parallel, sequence, race.

## Cancellation

Fiber can cancel itself. Another fiber can cancel a fiber. This stops execution after the current effect. Stops immediately for effects which support it.

How to avoid bad states when your fiber is cancelled. Uncancelable.

# Fault Tolerance

Timeout. 