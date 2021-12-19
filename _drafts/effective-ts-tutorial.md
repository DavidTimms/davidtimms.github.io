---
layout: post
title: Effective.ts Tutorial
date: 2021-12-19 14:00:00 +0000
categories: typescript effective.ts
---

[Effective.ts](https://github.com/DavidTimms/effective.ts) is a library I have implemented for writing safe, concurrent, fault-tolerant programs in TypeScript. It takes heavy inspiration from the way side effects are handled in pure functional systems like [Haskell](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/IO) and [Cats Effect (Scala)](https://typelevel.org/cats-effect/). This tutorial introduces some of the basic concepts and unique features of the library.

## Part one: Introducing IO

The most important concept in the library is a type called `IO`. An `IO` is an action - a fragment of a program. When it is run, it can perform side effects, like writing to disk or making HTTP requests. It is a thin wrapper around an asynchronous function, which is why it returns a promise when it is run.

The best way to write programs with effective.ts is to combine these small fragments of programs together into larger more complex actions, until you have a single `IO` which represents your entire program. That means you only need one call to `.run()` which comes at very top level of the program.

The `IO` type has two type parameters:

- `A` is the type of values returned by running the action.
- `E` is the type of errors raised by this action. For now, we can omit this, and let it default to `unknown`. We'll see more about handling errors in part two.

Let's see how we create an `IO`.

To get started we need to import `IO`:

```typescript
import { IO } from 'effective.ts';
```

The simplest way is using [`IO.wrap`](https://davidtimms.github.io/effective.ts/modules/io.IO.html#wrap). This creates an action which will return a specific value when it is run. The action always succeeds and does not perform any side effects. Some other libraries call this function `pure` or `return`.

```typescript
const return123: IO<number> = IO.wrap(123);

return123.run(); // return a promise which resolves to 123.
```

That's all well and good, but we want to perform side effects! For that we need to use [the `IO` function](https://davidtimms.github.io/effective.ts/modules/io.html#IO-2).

```typescript
const printHi: IO<void> = IO(() => console.log("Hi!"));

printHi.run(); // prints "Hi!" to the console.
```

In the above example, `console.log` does not return any useful value, so the result type for the action is `void`.

We can also create actions which perform asynchronous effects, such as network requests.

```typescript
import fetch from "node-fetch";

const makeRequest: IO<Response> =
    IO(() => fetch("https://www.wikipedia.org"));

// returns promise which resolves to a Response
// object once the request is complete.
makeRequest.run(); 
```

Notice that the type of `makeRequest` is `IO<Response>`, not `IO<Promise<Response>>`? That demonstrates something important - all actions in Effective.ts are implicitly asynchronous, so the types do not distinguish between synchronous and asynchronous. You shouldn't need to use `async` and `await` at all, or deal with promises directly. Asynchronous actions are awaited automatically by the runtime.

Now that we know how to build a basic action, how do we go about combining these into full programs? `IO` has a few methods to help.

[The `map` method](https://davidtimms.github.io/effective.ts/classes/io.IOBase.html#map) can be used to transform the result of an action using a pure function. It is equivalent to the `map` method on an array.

```typescript
const mapped: IO<number> = IO.wrap(123).map(x => x + 1);

mapped.run(); // return a promise which resolves to 124.
```

[The `andThen` method](https://davidtimms.github.io/effective.ts/classes/io.IOBase.html#andThen) is used to perform one action after another. The function which produces the second action takes the result of the first action as an argument. This function is sometimes also called `flatMap` or `bind`. It is what makes the `IO` type a monad.

```typescript
import fetch from "node-fetch";

const fetchWikipediaHomepage: IO<string> =
    IO(() => fetch("https://www.wikipedia.org"))
    .andThen(response => IO(() => response.text()))

// returns promise which resolves with the text of the response.
fetchWikipediaHomepage.run();
```

These features alone are enough to build up many programs. But what about if things go wrong? It's time to look at how we can handle errors.

## Part two: Handling Errors

IO has a second type for the error. When calling outside code, the error is unknown. Use raise instead of throw. Use mapError and catch. castError.

## Part three: Concurrency

What are fibers? How to start a fiber. How to wait for a fiber's outcome.

Parallel, sequence, race.

## Part four: Cancellation

Fiber can cancel itself. Another fiber can cancel a fiber. This stops execution after the current effect. Stops immediately for effects which support it.

How to avoid bad states when your fiber is cancelled. Uncancelable.