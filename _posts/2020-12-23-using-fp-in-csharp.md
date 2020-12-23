---
layout: post
title: Using Functional Programming Principles in C#
tags: [functional-programming, c#]
comments: true
excerpt_separator: <!--more-->
---

For a number of years now, Functional Programming and its concepts have become increasingly significant to the software industry.

By applying functional principles to your projects, your resulting applications will be more predictable, reliable, and maintainable.

<!--more-->

However, you don't have to use a [purely functional language](https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Functional_languages) like Haskell or Elm. We can follow the principles and patterns of FP even in object oriented languages such as C#.

(Although as a side note, ever since the introduction of LINQ and Lambda expressions, C# has increasingly borrowed features from functionally pure languages)

What's more, understanding the theory of FP concepts is not too difficult. In Robert C Martin's [*Clean Code*](https://www.amazon.co.uk/dp/0132350882/) (a book I'm guessing many developers have read), "Uncle Bob" advocates:
- Keeping functions small
- Not repeating code (the DRY principle)
- Only doing one thing
- Avoiding side effects, and
- Only having one level of abstraction

All of the above come naturally when programming in a functional style, even though the examples in *Clean Code* are written in Java.

In other words, the difference between OO and FP boils down to how you compose your application rather than the features of the language or framework.


## Using Expressions and Reducing Side Effects

In statement-based languages like C#, many language constructs do not produce a direct result, but are instead executed for a side effect. This is the opposite of FP which is expression-based, i.e. everything produces some result.

To illustrate, let’s look at the following code snippets.

```c#
//Statement based
string numberType;

if (number % 2 == 0)
{
    numberType = "Even";
}
else
{
    numberType = "Odd";
}
```

```c#
//Expression based
var numberType = (number % 2 == 0) ? "Even" : "Odd";
```

The first snippet uses the `if..else` statement to assign a value to the `numberType` string, whereas the second snippet uses the ternary operator expression.

You've probably noticed that the second snippet is shorter, which does tend to happen in FP, but the more important difference is that we no longer have an unassigned variable, nor do we have multiple places where `numberType` is assigned.

Now imagine that we need to log the value of numberType.

```c#
string numberType;

if (number % 2 == 0)
{
    numberType = "Even";
}
else
{
    numberType = "Odd";
}

var logMessage = $"{number} is {numberType}";
```

Note, that we have no choice but to assign `numberType` so that it can be used in the interpolated string. But it probably won’t get used again outside of this context.

By contrast, it's easy to reduce the expression version to a single expression. One of the nicest things about expressions is that they are naturally composable - expressions return a value, so they can easily be combined with other expressions or statements by using them in place of a variable or other value. For example:

```c#
var logMessage = $"{(number % 2 == 0 ? "Even" : "Odd")}";
```

Here, we are keeping all relevant information together and have eliminated the extraneous variable.

