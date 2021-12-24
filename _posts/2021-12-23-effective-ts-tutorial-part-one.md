---
layout: post
title: Effective.ts Tutorial - Part One
author: David Timms
categories: typescript effective.ts
license: null
---

[Effective.ts](https://github.com/DavidTimms/effective.ts) is a library I recently released, which aims to make it easier to write safe, concurrent, fault-tolerant programs in TypeScript. It takes heavy inspiration from the way side effects are handled in pure functional systems like [Haskell](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/IO) and [Cats Effect (Scala)](https://typelevel.org/cats-effect/). This tutorial introduces some of the basic concepts and unique features of the library.

## Introducing IO

The most important concept in the library is a type called `IO`. An `IO` is an action - a fragment of a program. When it is run, it can perform side effects, like writing to disk or making HTTP requests. It is a thin wrapper around an asynchronous function, which is why it returns a promise when it is run.

The best way to write programs with effective.ts is to combine these small fragments of programs together into larger more complex actions, until you have a single `IO` which represents your entire program. That means you only need one call to `.run()` which comes at very top level of the program.

At this stage, it might seem like unnecessary indirection to create an `IO` then run in later in the program, but this separation gives us much more power to manipulate the control flow of our program, as we will see in part two.

The `IO` type has two type parameters:

- `A` is the type of values returned when the action succeeds.
- `E` is the type of errors raised when the action fails. For now, we can omit this, and let it default to `unknown`. We'll see more about handling errors later.

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
import fetch, { Response } from "node-fetch";

const makeRequest: IO<Response> =
    IO(() => fetch("https://www.wikipedia.org"));

// returns promise which resolves to a Response
// object once the request is complete.
makeRequest.run(); 
```

Notice that the type of `makeRequest` is `IO<Response>`, not `IO<Promise<Response>>`? That demonstrates something important - all actions in effective.ts are implicitly asynchronous, so the types do not distinguish between synchronous and asynchronous. You shouldn't need to use `async` and `await` at all, or deal with promises directly. Asynchronous actions are awaited automatically by the runtime.

## Combining Actions

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

These features alone are enough to build up many programs. But in the real world, nothing ever goes quite to plan. It's time to look at how we can handle those pesky errors.

## Handling Errors

The standard way of dealing with errors in TypeScript is to throw an exception, then catch it with a `try/catch` block. While this does work, it has a few problems. It is hard to maintain type safety in `catch` blocks, because the compiler does not know what kinds of exception can be thrown. That is why TypeScript uses the rather unhelpful `any` or `unknown` types for caught exceptions.

The second problem comes when you call a library function. As a user of the library, it is hard to know what kinds of exceptions the function might throw. If you are lucky, this information will be described in the library documentation, and if you are _very_ lucky, the description will be up-to-date!

In effective.ts, you can "raise" an error, which is equivalent to throwing an exception. The difference is that the `IO` type describes in its second type parameter what types of error the action can possibly raise. This helps to make functions self-documenting.

```typescript
// Because the error type is "never", we know this
// action will never raise an error.
const neverFails: IO<number, never> = IO.wrap(456);
```

```typescript
const alwaysFails: IO<never, TypeError> =
    IO.raise(TypeError("validation failed"));

// Returns a promise which rejects with the TypeError.
alwaysFails.run();
```

As we saw earlier, when we want to call outside code with side effects, we need to wrap the call in `IO`. This includes any code which might throw an exception. When we run the action, it will catch any exceptions which are thrown and raise them in the `IO` context. Since we cannot know what exceptions third-party code might throw, the error type in `IO` type defaults to `unknown`.

```typescript
const printGreeting: IO<void, unknown> =
    IO(() => console.log("Greetings!"));
```

If you do not care about precise tracking of errors, you can leave them typed as `unknown` throughout the program. If you do want precision though, you can convert the unknown error to a known type. You can do this with [the `mapError` method](https://davidtimms.github.io/effective.ts/classes/io.IOBase.html#mapError), which is the calamitous cousin of the `map` method we used previously - it transforms an error using a pure function. If the action does not raise an error, the transformer function is never called.


```typescript
class PrintingError extends Error {
  constructor(readonly cause: unknown) {
    super(`Failed to print to console: ${cause}`);
  }
}

const knownError: IO<void, PrintingError> =
    printGreeting.mapError(error => new PrintingError(error));
```

If we want to recover from an error instead of just transforming it, we need [the `catch` method](https://davidtimms.github.io/effective.ts/classes/io.IOBase.html#catch). The catcher function returns an action, so it can use `IO.raise` to re-raise an error or `IO.wrap` to return a fallback value. 

```typescript
import fetch from "node-fetch";

const fetchWithCatch: IO<string, never> =
    IO(() => fetch("https://www.wikipedia.org"))
    .map(() => "fetched successfully")
    .catch(error => IO.wrap("failed to fetch"));

// returns a promise which resolves with
// "fetched successfully" or "failed to fetch".
fetchWithCatch.run();
```

In the example above, we can see that the error type for the entire `fetchWithCatch` action is `never`, meaning it will never raise an error. This is inferred because we caught any errors and did not re-raise them.

When programming with effective.ts, it is important to use `IO.raise` instead of throwing exceptions directly, otherwise the types of the errors will not be tracked correctly.

```typescript
const unsafelyThrow: IO<Response> =
    IO(() => fetch("https://www.wikipedia.org"))
    .andThen(response => {
        if (response.status >= 400) {
            // BAD
            throw Error("Request failed");
        } else {
            return IO.wrap(response);
        }
    });

const safelyRaise: IO<Response> =
    IO(() => fetch("https://www.wikipedia.org"))
    .andThen(response => {
        if (response.status >= 400) {
            // GOOD
            return IO.raise(Error("Request failed"))
        } else {
            return IO.wrap(response);
        }
    });
```

---

We've now covered the basics of effective.ts. We can use `IO` to represent actions and combine these actions into full programs with error handling.

In part two, we'll dive in to what really makes the library special - its support for concurrency, fault-tolerance and cancellation.
