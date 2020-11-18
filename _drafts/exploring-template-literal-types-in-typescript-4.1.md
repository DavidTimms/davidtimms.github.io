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

## Dispatching Actions

It's a common pattern in many applications to represent actions in the system as values. This lets you easily track history, pass the actions over a network, and make code easier to test. The types to represent our actions might look something like this.

```ts
type BakeBrownies = { tag: "BAKE_BROWNIES", quantity: number };
type EatBrownie = { tag: "EAT_BROWNIE" };
type Action = BakeBrownies | EatBrownie;
```

Eventually we need to do something with these action objects, so let's write a simple class to handle them.

```ts
class BrownieCounter {
    brownieCount = 0;

    bakeBrownies(action: BakeBrownies): void {
        this.brownieCount += action.quantity;
    }

    eatBrownie(action: EatBrownie): void {
        if (this.brownieCount > 0) {
            console.log("Om nom nom");
            this.brownieCount -= 1;
        } else {
            console.log("Noooooo!");
        }
    }
}
```

To glue them together, we'll need a function which takes the action and passes it to the correct method of our `BrownieCounter`.

```ts
function dispatchAction(brownieCounter: BrownieCounter, action: Action): void {
    switch (action.tag) {
        case "BAKE_BROWNIES":
            brownieCounter.bakeBrownies(action);
            break;
        case "EAT_BROWNIE":
            brownieCounter.eatBrownie(action);
            break;
    }
}
```

This is ok for now, but as we add more actions to our program, it's going to get annoying having to add a new case for every one. Since the method names on our `BrownieCounter` are just camel case versions of the action tags, we can rewrite `dispatchAction` to generate the method name and call it. We just need a little `snakeCaseToCamelCase` helper function.

```ts
function dispatchAction(brownieCounter: BrownieCounter, action: Action): void {
    (brownieCounter as any)[snakeCaseToCamelCase(action.tag)](action);
}

function snakeCaseToCamelCase(snakeCaseString: string): string {
    return (
        snakeCaseString
        .split("_")
        .map((word, i) =>
            i === 0 ?
                word.toLowerCase() :
                word && (word[0].toUpperCase() + word.slice(1).toLowerCase())
        )
        .join("")
    );
}
```

This avoids the boilerplate, but it's a step backwards for type safety. As the type signature of `snakeCaseToCamelCase` only says it returns a string, the compiler has no way to know whether that string will be a valid method name for `BrownieCounter`, so we have to cast to `any` to allow it. Even worse, if we add a new action to the program, but forget to add a method to `BrownieCounter` to handle it, we won't get a type error. It will not be until runtime that we see this sad message.

> `brownieCounter[snakeCaseToCamelCase(...)] is not a function`

Luckily, with TypeScript 4.1, we no longer need to chose between boilerplate and type safety. Using template literal types we can reimplement our case conversion at the type level. This requires another handy new feature - four new types for manipulating string literal types called `Uppercase`, `Lowercase`, `Capitalize` and `Uncapitalize`.

```ts
type SnakeCaseToCamelCase<S extends string> =
    S extends `${infer FirstWord}_${infer Rest}` ?
        `${Lowercase<FirstWord>}${SnakeCaseToPascalCase<Rest>}` :
        `${Lowercase<S>}`;

type SnakeCaseToPascalCase<S extends string> =
    S extends `${infer FirstWord}_${infer Rest}` ?
        `${Capitalize<Lowercase<FirstWord>>}${SnakeCaseToPascalCase<Rest>}` :
        Capitalize<Lowercase<S>>;

type BakeBrowniesCamelCase = SnakeCaseToCamelCase<"BAKE_BROWNIES">;
// = "bakeBrownies"
```

We can use this fancy new type to provide a more informative signature for our helper function.

```ts
function snakeCaseToCamelCase<S extends string>(
    snakeCaseString: S,
): SnakeCaseToCamelCase<S> {
    return (
        snakeCaseString
        .split("_")
        .map((word, i) =>
            i === 0 ?
                word.toLowerCase() :
                word && (word[0].toUpperCase() + word.slice(1).toLowerCase())
        )
        .join("")
    ) as SnakeCaseToCamelCase<S>
}

const tag = snakeCaseToCamelCase("BAKE_BROWNIES");
// has type "bakeBrownies"
```

This lets us remove the unsafe type cast from `dispatchAction`.

```ts
function dispatchAction(brownieCounter: BrownieCounter, action: Action): void {
    brownieCounter[snakeCaseToCamelCase(action.tag)](action as any);
}
```

Unfortunately we still need to cast the *action* to `any`, as the compiler isn't quite smart enough to figure out that we've picked the *right* method for handling the action. Still, it's a big improvement. If we forget to add a method to `BrownieCounter` for a new action, we'll get a nice clear error message pointing out our mistake.

```ts
type StealBrownie = { tag: "STEAL_BROWNIE" };

type Action = BakeBrownies | EatBrownie | StealBrownie;
```

> `Property 'stealBrownie' does not exist on type 'BrownieCounter'.`

Success!

## A SQL Database

These small examples are all well and good, but surely you couldn't use this technique to interpret something as complex as a SQL query?

Actually, you could. I know because [Charles Pick of codemix has done just that.](https://github.com/codemix/ts-sql) It's a brilliant demonstration of the power of the type system, but I wouldn't go uninstalling PostgreSQL just yet.

## That's All Folks


I hope you enjoyed this dive into the world of template literal types.

If you've spotted any errors in this post, [please let me know by opening an issue on GitHub.](https://github.com/DavidTimms/davidtimms.github.io/issues/new)