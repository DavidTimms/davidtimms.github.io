---
layout: post
title: Seven Days of OCaml
date: 2021-12-30 16:00:00 +0000
categories: programming-languages ocaml
---

A couple of weeks ago, I returned from an ill-advised excursion to London to discover that I had picked up COVID-19. Whoops. Luckily my symptoms were very mild, but I still had to remain in self-isolation for ten days. What better way to fill this gaping void of time than learning a new programming language by tackling some [Advent of Code](https://adventofcode.com) puzzles.

[OCaml](https://ocaml.org) is a language which has been on my radar for a few years, particularly since listening to the excellent [Signals and Threads](https://signalsandthreads.com) podcast from Jane Street. I wanted to compare it to the other functional languages I have used, Scala and Haskell. The idea of an mature ML family language which is more focussed on pragmatism than purity and category theory appealed to me.[^dont-shoot-me]

As somebody who rather enjoys _implementing_ [programming](https://github.com/DavidTimms/loxdown) [languages](https://github.com/DavidTimms/sonata), I also wanted to see whether OCaml would be a good candidate for future projects in that space. Its combination of language features, performance and libraries make it particularly suited to writing compilers. Graydon Hoare chose it for the original [Rust](https://www.rust-lang.org) compiler and Facebook for the [Hack](https://hacklang.org) compiler.

I completed the first seven puzzles in this year's calendar. I am still very much a novice - experienced OCamlers would cringe at [my code](https://github.com/DavidTimms/advent-of-code-2021-ocaml), I'm sure. It now feels like I've seen enough to form an initial opinion though.

## The Good

**The OCaml compiler is _fast_**. Admittedly, my program was not very large, but even after pulling in a couple of sizable libraries from [opam](https://opam.ocaml.org/), builds are near instant. Under a second for a clean build and around 100ms for an incremental build. After using TypeScript and Scala for a couple of years, this is a fresh of fresh air. The faster feedback loop makes the development experience much more pleasant. As with other statically-typed languages, I do find that I need to run my code less frequently while developing than I do in dynamic languages. That means the fast compile times are merely a nice-to-have, rather than an essential part of the workflow.  

Another aspect of the developer experience which I enjoyed was **[the VSCode plugin](https://marketplace.visualstudio.com/items?itemName=ocamllabs.ocaml-platform)**. This works well out-of-the-box, and provides responsive type-checking as you type. I appreciate the type annotations it provides above `let` declarations, particularly when relying on the inferred parameter and return types instead of specifying them explicitly.

Not only is the compiler faster than many of its peers, but its Hindley-Milner type inference algorithm is the most effective I have ever used. It feels almost magical to write a non-trivial program with no type annotations and have the compiler derive precise and correct static types throughout. In practice, I found that I usually did want to explicitly provide parameter and return types, since I prefer to start writing a function by thinking about the inputs and outputs, then tackle the implementation later.

Having written **functional-style code** in languages where it is not the primary paradigm (TypeScript, Python), it feels great to use a language like OCaml which makes those patterns easy and elegant. The built-in immutable list, map and set data structures meant I didn't need to worry about accidental mutation or cloning. They are a joy to manipulate with higher-order functions (e.g. `map`, `filter`, `fold`, `sum`, `find`), especially when combined with universal [currying](https://en.wikipedia.org/wiki/Currying). I'm still not 100% sold on the merit of curried functions though - they can certainly lead to some beautiful and concise code, but as you get further towards full [point-free style](https://en.wikipedia.org/wiki/Tacit_programming), it can become impenetrably dense. I would rather a function which is twice as long, if it takes half the time to understand. As always, good judgement and a little restraint go a long way.

OCaml's **pattern matching** is simple, but effective. Modelling data with [ADTs](https://en.wikipedia.org/wiki/Algebraic_data_type) then manipulating them with pattern matching functions is just such a good fit for the types of puzzles in Advent of Code. In Scala, I have found that this style can be just as effective on more complex "real world" problems, and I look forward to tackling some of those in OCaml in future.

The language's **syntax** was something I did not expect to like. Facebook thought OCaml's syntax was so unintuitive that they created [an alternative syntax, Reason](https://en.wikipedia.org/wiki/Reason_(programming_language)), to make it look more like JavaScript. The syntax is certainly unfamiliar for anybody used to [C family languages](https://en.wikipedia.org/wiki/Category:C_programming_language_family), but I found it quite quick to pick up. The core of the language is quite small, and once I had exorcised my TypeScript muscle memory, I had few problems. Where I did make mistakes, they were picked up by the compiler. In a statically-typed language it is quite hard to make a syntax error which leads to unexpected behaviour at runtime.

I am a big fan of **optional and named parameters**, so it was a nice surprise to find both present in OCaml. I think these features are a big part why I find OCaml easier to read than Haskell. I was using Jane Street's alternative standard library, [base](https://opam.ocaml.org/packages/base/), which adds a ton of useful functions, and uses named parameters very heavily. Personally, I think they are slightly overused there, as explicit names at the call site don't add much clarity to generic functions like `List.map`.

I was glad to see **[the pipeline operator](https://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html) (`|>`)** is present in OCaml. As far as I am aware, this first appeared in F♯, with variations having since appeared in Elixir, Hack, and [perhaps soon JavaScript](https://github.com/tc39/proposal-pipeline-operator/). This is such a small addition to the language, but makes a big difference to the experience of using it. It is natural to view a function as a serious of steps, laid out top to bottom, left to right, instead of having to read backwards from the most deeply nested expression. This is why "fluent" method chaining interfaces are so popular in object-oriented languages. Like pattern matching, this feels like a feature which will become mainstream over the next few years.

## The Bad

While OCaml is a mature and battle-tested language, its **ecosystem is small and fractured**. There are some good learning resources available, such as [Real World OCaml](https://dev.realworldocaml.org/), there are far fewer than for more popular languages. This may be unfounded, but I worry that once I get in to the more obscure areas of the language, finding answers and explanation online will become difficult. This isn't helped by having two [competing](https://opam.ocaml.org/packages/base/) [standard libraries](https://ocaml.org/api/Stdlib.html), two [competing syntaxes](https://reasonml.github.io/), and part of the community apparently forking to create [a new language](https://rescript-lang.org/).

When it comes to **libraries**, OCaml's package manager [lists just 3,668](https://opam.ocaml.org/packages/) packages - a far cry from the [1,841,493 JavaScript packages](https://www.npmjs.com/) or [349,026 Python packages](https://pypi.org/) available. Obviously, quality is more important than quantity, but it does indicate that you should be prepared to write your own libraries for some areas if you choose OCaml for a serious project.

I must confess that when debugging, I quite often fall back on the tried and true technique of littering the code with print calls. I found this curious hard to do in OCaml, since there is **no universal string representation for data structures**. I wanted to be able to print out intermediate maps, lists, sets and records to verify that my parsing was working correctly, but I was not able to find a way to do this, other than writing repetitive functions to produce string representations. Perhaps I need to find a more sophisticated method of debugging, but I longed to be able to `.toString()` as I would in Scala or TypeScript.

Lack of ad hoc and parametric polymorphism.

## The Untouched

GADTs.

PPX.

Procedural programming.

Object-oriented programming.

## Conclusion

Like OCaml. It badly needs type classes / implicits. Would use it for a language implementation.

[^dont-shoot-me]: For the record, I think Haskell is a wonderful language. Category theory is a great tool for sculpting composable abstractions, but it is not a goal in and of itself.