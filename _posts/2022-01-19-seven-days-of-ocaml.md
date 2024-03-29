---
layout: post
title: Seven Days of OCaml
author: David Timms
categories: programming-languages ocaml
license: null
---

A few weeks ago, I returned from an ill-advised excursion to London to discover that I had picked up COVID-19. Whoops. Luckily my symptoms were very mild, but I still had to remain in self-isolation for ten days. What better way to fill this gaping void of time than learning a new programming language by tackling some [Advent of Code](https://adventofcode.com) puzzles?

[OCaml](https://ocaml.org) is a language which has been on my radar for a few years, particularly since listening to the excellent [Signals and Threads](https://signalsandthreads.com) podcast from Jane Street. I wanted to compare it to the other functional languages I've used, Scala and Haskell. The idea of an mature language in the ML family which is more focussed on pragmatism than purity and category theory appealed to me.

As somebody who rather enjoys _implementing_ [programming](https://github.com/DavidTimms/loxdown) [languages](https://github.com/DavidTimms/sonata), I also wanted to see whether OCaml would be a good candidate for future projects in that space. Its combination of language features, performance and libraries make it particularly suited to writing compilers. That's why Graydon Hoare chose it for the original [Rust](https://www.rust-lang.org) compiler, and Facebook for the [Hack](https://hacklang.org) compiler.

I only completed the first seven puzzles in this year's calendar, but it now feels like I've seen enough to form an initial opinion though. I am still very much a novice though - experienced OCamlers would cringe at [my code](https://github.com/DavidTimms/advent-of-code-2021-ocaml), I'm sure. 

## The Good

**The OCaml compiler is _fast_**. Admittedly, my program was not very large, but even after pulling in a couple of sizable libraries from [opam](https://opam.ocaml.org/), builds are near instant - under a second for a clean build, and around 100ms for an incremental build. After using TypeScript and Scala for a couple of years, this is a fresh of fresh air. The faster feedback loop makes the development experience much more pleasant. As with other statically-typed languages, I do find that I need to run my code less frequently while developing than I do in dynamic languages. That means the fast compile times are merely a nice-to-have, rather than being essential for productivity, but they are welcome nonetheless.

Another aspect of the developer experience which I enjoyed was **[the VSCode plugin](https://marketplace.visualstudio.com/items?itemName=ocamllabs.ocaml-platform)**. This works well out-of-the-box, and provides responsive type-checking as you type. It can display type annotations above `let` declarations, which I found particularly useful when relying on the inferred types instead of specifying them explicitly.

Not only is the compiler faster than many of its peers, but its **[Hindley-Milner type inference algorithm](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)** is the most effective I have ever used. It feels almost magical to write a non-trivial program with no type annotations and have the compiler derive precise and correct static types throughout. In practice, I found that I usually did want to explicitly provide parameter and return types, but for my benefit, not the compiler's. I prefer to write a function by thinking about the input and output types first, then tackling the implementation later.

Having written **functional-style code** in languages where it is not the primary paradigm (TypeScript, Python), it feels great to use a language like OCaml which makes those patterns easy and elegant. The built-in immutable list, map and set data structures meant I didn't need to worry about accidental mutation or cloning. They are a joy to manipulate with higher-order functions (e.g. `map`, `filter`, `fold`, `sum`, `find`), especially when combined with universal [currying](https://en.wikipedia.org/wiki/Currying). I'm still not 100% sold on the merits of curried functions though - they can certainly lead to some beautiful and concise code, but as you get further towards full [point-free style](https://en.wikipedia.org/wiki/Tacit_programming), it can become impenetrably dense. I would rather a function which is twice as long, if it takes half the time to understand. As always, good judgement and a little restraint go a long way.

OCaml's **pattern matching** is simple, but effective. Modelling data with [ADTs](https://en.wikipedia.org/wiki/Algebraic_data_type) then manipulating them with pattern matching functions is such a good fit for the types of puzzles in Advent of Code. In Scala, I have found that this style can be just as effective on more complex "real world" problems, and I look forward to tackling some of those in OCaml in future.

The language's **syntax** was something I did not expect to like. Facebook thought OCaml's syntax was so unintuitive that they created [an alternative syntax, Reason](https://en.wikipedia.org/wiki/Reason_(programming_language)), to make it look more like JavaScript. The syntax is certainly unfamiliar for anybody used to [C family languages](https://en.wikipedia.org/wiki/Category:C_programming_language_family), but I found it easy to pick up. The core of the language is quite small, and once I had exorcised my TypeScript muscle memory, I had few problems. Where I did make mistakes, they were picked up by the compiler. In a statically-typed language, it is actually quite hard to make a syntax error which leads to unexpected behaviour at runtime.

Having spent years writing Python, I am a big fan of **optional and named parameters**, so it was a nice surprise to find both present in OCaml. I think these features are a big part why I find OCaml easier to read than Haskell. I was using Jane Street's alternative standard library, [base](https://opam.ocaml.org/packages/base/), which adds a ton of helpful functions, and uses named parameters very heavily. Personally, I think they are slightly overused there. Explicit names at the call site don't add much clarity to generic functions like `List.map`.

I was glad to see that **[the pipeline operator](https://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html) (`|>`)** is present in OCaml. As far as I am aware, this first appeared in [F♯](https://fsharp.org), with variations having since appeared in [Elixir](https://elixir-lang.org), [Hack](https://hacklang.org), and [perhaps soon JavaScript](https://github.com/tc39/proposal-pipeline-operator/). This is such a small addition to the language, but makes a big difference to the experience of using it. It is natural to view a function as a serious of steps, laid out top to bottom, left to right, instead of having to read backwards from the most deeply nested expression. This is why "fluent" method chaining interfaces are so popular in object-oriented languages. Just like pattern matching, this feels like a feature which will become mainstream over the next few years as more languages adopt some variation of it.

## The Bad

While OCaml is a mature and battle-tested language, its **ecosystem is small and fractured**. There are some good learning resources available, such as [Real World OCaml](https://dev.realworldocaml.org/), but far fewer than for more popular languages. This may be unfounded, but I worry that once I get in to the more obscure areas of the language, finding answers and explanation online will become difficult. This isn't helped by having two [competing](https://opam.ocaml.org/packages/base/) [standard libraries](https://ocaml.org/api/Stdlib.html), two [competing syntaxes](https://reasonml.github.io/), and part of the community apparently forking to create [a new language](https://rescript-lang.org/).

When it comes to **libraries**, OCaml's package manager [lists just 3,668](https://opam.ocaml.org/packages/) packages - a far cry from the [1,841,493 JavaScript packages](https://www.npmjs.com/) or [349,026 Python packages](https://pypi.org/) available. Obviously, quality is more important than quantity, but it does indicate that you should be prepared to write your own libraries for some areas if you choose OCaml for a serious project.

When debugging, I must confess that I quite often fall back on the tried and true technique of littering the code with print calls. I found this curious hard to do in OCaml, since there is **no universal string representation for data structures**. I wanted to be able to print out intermediate maps, lists, sets and records to verify that my parsing was working correctly, but I was not able to find a way to do this, other than writing repetitive functions to produce string representations. Perhaps I need to find a more sophisticated method of debugging, but I longed to be able to call an equivalent of `.toString()` as I would in Scala or TypeScript.

My frustration with converting to strings is really a symptom of a much wider limitation - the **lack of ad-hoc polymorphism**. Other languages have different features to solve this problem - Haskell has type classes, Rust has traits, Scala has implicits. These all work in a similar way - by allowing operations on a type to be looked up and passed to a function implicitly by the compiler. This allows generic functions to do useful things with the parameters they are passed. For instance, producing string representations of data types in Haskell is elegantly handled by [the `Text.Show` type class](https://hackage.haskell.org/package/base-4.16.0.0/docs/Text-Show.html). As OCaml has no way to do this, polymorphic functions must be _fully_ polymorphic - able to operate on _any_ concrete type. The most commonly suggested workaround is to explicitly provide a first-class module. This does work, but it requires quite a bit of boilerplate and clutters the call site. I found the standard library map and set types far harder to work with in OCaml than they are in other languages due to this.

Most object-oriented languages tackle the problem by using some form of subtyping, such as interfaces, inheritance or structural types. This is a bit less flexible than type classes, but it works well enough for most use cases. While OCaml has a very capable object system, it does not feel well unified with the rest of the language. Integers are not objects. Lists are not objects. Records are not objects. This means there is no "iterable" interface for collections and no "comparable" interface for types which can be used in sets.

The maintainers of OCaml are well aware of this shortcoming, and aim to eventually add [modular implicits](https://arxiv.org/pdf/1512.01895.pdf) to the language. In recent years, it seems to have been deprioritised in favour of [multicore and algebraic effects](https://discuss.ocaml.org/t/multicore-ocaml-september-2021-effect-handlers-will-be-in-ocaml-5-0/8554) though. I really hope that modular implicits make it in eventually, as they would hugely improve the language's ergonomics. 

## The Untouched

There are still many features of OCaml which I have not yet had an opportunity to use. These include some of its most interesting and distinct aspects, so I hope I do get a chance to explore them in a future project. Perhaps some of these could help with the pain points discussed above. The ones I am aware of are:

- [Metaprogramming and PPX](https://ocamlverse.github.io/content/metaprogramming.html)
- [GADTs](https://ocaml.org/manual/gadts.html)
- [Functors and other advanced uses of the module system](https://ocaml.org/learn/tutorials/functors.html)
- [Classes and objects](https://ocaml.org/learn/tutorials/objects.html)

## Conclusion

I thoroughly enjoyed my time using OCaml. The language feels mature, approachable and productive. While I still wish it had type classes, I'm sure I'll find better ways to cope without them as I explore further. I'm keen now to move on to some "real-world" projects - perhaps another compiler, or a web app, or a CLI tool.

If you're interested in functional programming, or, like me, you just want to give a new language a go, why not take OCaml for a spin? You might like it too.
