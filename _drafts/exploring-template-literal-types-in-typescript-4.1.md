---
layout: post
title: Exploring Template Literal Types in TypeScript 4.1
categories: programming-languages typescript
---

[TypeScript 4.1 has just been released.](https://devblogs.microsoft.com/typescript/announcing-typescript-4-1-beta/) This edition builds on the features [introduced in version 4.0](https://devblogs.microsoft.com/typescript/announcing-typescript-4-1-beta/) with a group of new tools which provide typesafe ways to express common dynamic JavaScript patterns. As with most powerful type system features, they can also be used in weird and wonderful ways, with bizarre results.

One of the most interesting new features is template literal types. Back in ES2015, a new string syntax called a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) was added to JavaScript, providing convenient and readable way to insert values into strings. These strings are surrounded by backticks and evaluate any expression surrounded by `${` ... `}`, inserting the result into the string.

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

