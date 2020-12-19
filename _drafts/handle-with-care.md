---
layout: post
title: Handle with Care
date: 2020-08-03 21:20:04 +0100
categories: programming-languages
---
- This post will be about error handling.

- Usual goals of error handling:
  - Stop execution
  - Provide useful information for the user or developer
  - Leave the system in a valid state

- What about programs which carry on after errors?
  - Mention Loxdown, Load and Extract
  - Why unchecked exceptions are a poor fit for this:
    - Not clear which things can go wrong
    - Hide errors from the control flow
  - Functional error handling:
    - Result/Either
    - Validated
  - Error handling in Loxdown typechecker:
    - Errors array as mutable state
      - Sorting errors by source location
    - PreviousTypeError class
    - Challenges
      - Ensuring all code checks for PreviousTypeError
      - Errors as a side-effect mean avoiding typechecking a node twice. This is both good and bad.
      - TypeChecker became a God object
    
