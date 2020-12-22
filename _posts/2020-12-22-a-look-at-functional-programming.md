---
layout: post
title: A Brief Look at Functional Programming
tags: [functional-programming]
excerpt_separator: <!--more-->
---

I’ve been reading about Functional Programming for many years, but as I don’t code in a functional language such as F# or Haskell, I’ve not really had the chance to see how I can make use of FP principles.

Fortunately, as work slows down for the Christmas holidays I’ve now found some spare time to revisit this topic. In this post I'll give a general overview of FP, and then in future posts I hope to describe how to apply FP principles to C# (my primary language).

<!--more-->

## What is Functional Programming?

Like many programming concepts, definitions are often subjective. Personally, I thought this definition by [Akhil Bhadwal](https://hackr.io/blog/functional-programming) is accurate and concise:

> Functional programming is a programming paradigm in which it tries to bind everything in pure mathematical functions. It is a declarative type of programming style that focuses on what to solve rather than how to solve.

But what is a "mathematical function"? It's simply a mapping from an input to an output. This is opposed to functions typically seen in non-FP programming languages such as C# or Javascript, where the function represents an encapsulated sequence of instructions that performs an action or changes the state of the object.

By applying FP principles we’ll quickly identify the separation between data and behaviour.

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

- **Functions are first class:** In the FP paradigm, functions are treated as variables; they can be passed as arguments or stored in data structures. 
- **Higher-order function:** This is a function that receives another function as an argument, or returns another function, or both.
- **Pure function:** This is a function that has no side-effects. For a particular input, it *always* returns the same output.
- **Referential transparency:** Because of the nature of pure functions, for a given input we can replace the function with its return value in all subsequent calls to that function.
- **Recursion:** There is no for and while loops in FP. Instead, we rely on recursion for iteration.
- **Variables are immutable:** It is not possible to modify a variable once it has been initialized. However, we can pass that variable to a function which returns another value.


## Advantages of Functional Programming

#### Controlling Side Effects

One major benefit of FP is the controlling of side effects. In programming, a side effect is anything that happens to the state of a system as a result of invoking a function.

Sometimes this is benign such as writing to a log, but quite often in OO-programing, we have to debug some unexpected behaviour because a method has changed some shared system state.

For example, let us compare the following code snippets:

```c#
public class Product
{
    public decimal Price { get; set; }

    public void UpdatePrice(decimal vat, decimal cost, decimal markup)
    {
        this.Price = (cost + markup) * (1 + vat);
    }

    // Rest of class
}
```

```c#
public decimal PriceIncludingVat(decimal vat, decimal cost, decimal markup)
{
    return (cost + markup) * (1 + vat);
}
```

In the `Product` class, `UpdatePrice` changes the value of `Price`. This is what is meant by a side-effect. It's a public method, and we may not be aware that another part of the system is calling it.

On the other hand, calling `PriceIncludingVat` doesn't modify existing state; it returns new state. You can call it as many times as you want and your price variable won't change until you actually assign it, e.g. `var price = PriceIncludingVat(0.2m, 10m, 2m);`


#### Easier to Understand

When a function has side effects, we not only need to consider its primary result, but also how it affects the rest of the system. This means that as a developer, at any one time, we need to be thinking about multiple system components - it's mentally exhausting!

However, because pure functions don’t change any states and are entirely dependent on the input, they are simple to understand.

Pure functions clear the way for making complex systems more predictable because they always behave the same way regardless of the external state.


#### Easier to Test and Debug

Side effects often also make functions more difficult to test, since their very nature implies dependencies on other parts of the system. Thus, no side effects means not having to worry about Mocks and Fakes.

Also, because FP uses immutable variables, it makes debugging easier as well.


#### Suited to Asynchronous Execution

Finally, because pure functions don't change the overall system state, they are naturally more suited for parallel or asynchronous execution.


## Disadvantages

- There is a learning curve to overcome (which I'm now finding!)
- Immutable values combined with recursion may lead to a performance reduction
- Using recursion instead of loops can be a daunting task
