---
layout: post
title: Exploring Template Literal Types in TypeScript 4.1
categories: programming-languages typescript
---

[TypeScript 4.1 has just been released.](https://devblogs.microsoft.com/typescript/announcing-typescript-4-1-beta/) This edition builds on the features [introduced in version 4.0](https://devblogs.microsoft.com/typescript/announcing-typescript-4-1-beta/) with a group of new tools which provide typesafe ways to express common dynamic JavaScript patterns. As with most powerful type system features, they can also be used and abused in weird and wonderful ways.

One of the most interesting new features is __template literal types__. Back in ES2015, a new string syntax called a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) was added to JavaScript, providing a convenient and readable way to insert values into strings. These are strings delimited by backticks which evaluate any expression surrounded by `${` ... `}`, inserting the result into the string.

```ts
const name = "Anders";
console.log(`Hello, ${name}!`); // prints "Hello, Anders!"
console.log(`I said, HELLO ${name.toUpperCase()}!`); // print "I said, HELLO ANDERS!"
```

In TypeScript 4.1, you can now use the template literal syntax in *types* as well as *values*. This means if you have a type which represents a single string literal or, more likely, a union of multiple string literals, you can use a template literal type to create a new string literal type derived from the old one.

```ts
type Animal = "shark" | "giraffe" | "platypus";
type BabyAnimal = `baby-${Animal}`; // = "baby-shark" | "baby-giraffe" | "baby-platypus"
```

As the example above shows, the transformation is distributed over the union, being applied to each member separately, then producing a union of the results. If you include multiple union types in the template literal, it produces every combination.

```ts
type Animal = "shark" | "giraffe" | "platypus";
type Age = "baby" | "adolescent" | "old";
type BabyAnimal = `${Age}-${Animal}`;
// = "baby-shark" |
//   "baby-giraffe" |
//   "baby-platypus" |
//   "adolescent-shark" |
//   "adolescent-giraffe" |
//   "adolescent-platypus" |
//   "old-shark" |
//   "old-giraffe" |
//   "old-platypus"
```

As well as building up new string literal types from shorter pieces, we can also extract parts of the string by matching a pattern. To do this we need to combine template literal types with [conditional types and the `infer` keyword](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-inference-in-conditional-types).

```ts
type ExtractVerb<S extends string> =
    S extends `${infer Verb} ${infer Activity}` ? Verb : never;

type Verbs = ExtractVerb<"play chess" | "write code" | "read hacker news">
// = "play" | "write" | "read"
```

This means we can break down a string into its substrings and return different types based on the contents of those substrings. That is everything we need to implement a simple parser. Let's look at some potential applications.

## Routing

In Node.js web frameworks like [Express](https://expressjs.com), it is common to define URL routing rules by using a path pattern given as a string. These strings can contain placeholders to represent dynamic parts of the path like IDs and slugs. Express uses a colon prefix for these path parameters. You can then access the values of these parameters in the route handler function using the object `req.params`.

```ts
app.get("/users/:userId/posts/:postId", (req, res) => {
    const userId = req.params.userId;
    const postId = req.params.postld;
    res.send(`Requested post ${postId} from user ${userId}`);
});
```

The code above will be perfectly acceptable to the TypeScript compiler, but if you run it, you'll notice the server responds with something like this:

> Requested post undefined from user 12

There is a typo in the function, but TypeScript did not complain. That is because the type of `req.params` is a `ParamsDictionary`.

```ts
interface ParamsDictionary {
    [key: string]: string;
}
``` 

This type just says it is an object with string keys and string values. It knows nothing about which keys the object will actually have at runtime, so it isn't much better than `any`. Template literal types allow us to parse the path to produce a safe and accurate type for `req.params`.

First, we need a generic type which can parse our path string literal to extract the parameters preceded by a colon. At the type level, we don't have normal data structures like arrays, so we will need to represent the parameters as a union.

```ts
type PathParams<Path extends string> =
    Path extends `:${infer Param}/${infer Rest}` ? Param | PathParams<Rest> :
    Path extends `:${infer Param}` ? Param :
    Path extends `${infer _Prefix}:${infer Rest}` ? PathParams<`:${Rest}`> :
    never;
```

It may look a little scary, but once you break it down, it's not too bad.

- The first case matches parameters at the start or middle of the path.  
  e.g. `":userId/posts/:postId"`.
- The second case matches parameters at the end of the path.  
  e.g. `":postId"`.
- The third matches strings with some other content before a parameter and strips it away.  
  e.g. `"/posts/:postId"`.
- In the fourth and final case where none of the three patterns match, we return the `never` type, because this behaves like an empty set when used like a union.

In the first and third cases it recursively calls itself with a shorter section of the string.

Applying this type to the path from above gives the parameter names, as we expect.

```ts
type ParamNames = PathParams<"/users/:userId/posts/:postId">;
// = "userId" | "postId"
```

We can use a mapped type to generate an object with those parameter names as keys.

```ts
type PathArgs<Path extends string> = {
    [K in PathParams<Path>]: string
};

type Params = PathArgs<"/users/:userId/posts/:postId">;
// = { userId: string, postId: string }
```

Great! That's the type we want for `req.params`.

Now all that is left to do is to use this type in the routing method of our web framework.

```ts
const app = {
    get<P extends string>(
        path: P,
        handler: (req: { params: PathArgs<P> }, res: any) => void,
    ): void {
        // web framework goes here...
    }
    // other methods...
};
```

*__NOTE:__ In a real implementation, `req` would have many other properties and `res` would be strongly typed. I have omitted those details to simplify this example.*

Now, the compiler catches the typo in our route handler. It even provides a suggestion to fix it!

```ts
app.get("/users/:userId/posts/:postId", (req, res) => {
    const userId = req.params.userId;
    const postId = req.params.postld;
    res.send(`Requested post ${postId} from user ${userId}`);
});
```

> `Property 'postld' does not exist on type 'PathArgs<"/users/:userId/posts/:postId">'. Did you mean 'postId'?`
