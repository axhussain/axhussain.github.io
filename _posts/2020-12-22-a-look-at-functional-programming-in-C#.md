---
layout: post
title: A Brief Look at Functional Programming in C#
tags: [functional-programming, c#]
excerpt_separator: <!--more-->
---

# A Brief Look at Functional Programming in C#

I’ve been reading about Functional Programming for many years, but as I don’t code in a functional language such as F# or Haskell, I’ve not really had the chance to see how I can make use of FP principles.

Fortunately, as work slows down for the Christmas holidays I’ve now found some spare time to revisit this topic.

<!--more-->

## What is Functional Programming?

First of all, the word "function" in "functional programming" is meant to refer to functions in the mathematical sense. In other words, a mapping from an input to an output.

Do not conflate mathematical functions with methods or functions typically seen in non-FP programming languages such as C# or Javascript, where the function represents an encapsulated sequence of instructions that performs an action or changes the state of the object.

In FP, we define functions in terms of their input and output, as opposed to how they change system state. By applying this principle we’ll quickly identify the separation between data and behavior.

<div align="center">
  <blockquote class="twitter-tweet" data-dnt="true" data-theme="light">
    <p lang="en" dir="ltr">
      OO makes code understandable by encapsulating moving parts. FP makes code understandable by minimizing moving parts.
    </p>
    &mdash; Michael Feathers (@mfeathers) <a href="https://twitter.com/mfeathers/status/29581296216?ref_src=twsrc%5Etfw">November 3, 2010</a>
  </blockquote>
  <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

### Some key FP concepts:

- **Higher-order function:** This is a function that receives another function as an argument, or returns another function, or both.
- **Pure function:** This is a function that has no side-effects. For a particular input, it *always* returns the same output.
- **Referential transparency:** Because of the nature of pure functions, for a particular input, we can replace the function or expression with its return value in all subsequent references to it.

## The Benefits of FP

3 themes from https://app.pluralsight.com/library/courses/functional-programming-csharp

1. Taming Side-Effects
2. FP is expression based - everything produces a result
3. Functions as data
