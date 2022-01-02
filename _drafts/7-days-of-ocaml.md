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

Functional features - immutable data structures, pattern matching, currying.

Syntax - unfamiliar but quite elegant. Named and optional parameters. Pipeline operator.

## The Bad

Lack of ad hoc and parametric polymorphism.

No easy way to print data structures for debugging.

Ecosystem is small and fractured.

## The Untouched

GADTs.

PPX.

Procedural programming.

Object-oriented programming.

## Conclusion

Like OCaml. It badly needs type classes / implicits. Would use it for a language implementation.

[^dont-shoot-me]: For the record, I think Haskell is a wonderful language. Category theory is a great tool for sculpting composable abstractions, but it is not a goal in and of itself.