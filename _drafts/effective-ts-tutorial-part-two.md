---
layout: post
title: Effective.ts Tutorial - Part Two
date: 2021-12-19 14:00:00 +0000
categories: typescript effective.ts
---

## Concurrency

What are fibers? How to start a fiber. How to wait for a fiber's outcome.

Parallel, sequence, race.

## Cancellation

Fiber can cancel itself. Another fiber can cancel a fiber. This stops execution after the current effect. Stops immediately for effects which support it.

How to avoid bad states when your fiber is cancelled. Uncancelable.