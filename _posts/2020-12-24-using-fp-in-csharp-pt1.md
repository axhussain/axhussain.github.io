---
layout: post
title: Using Functional Programming Principles in C# (Part 1)
tags: [functional-programming, c#]
comments: true
excerpt_separator: <!--more-->
---

Following on from my <a href="/2020/12/22/a-look-at-functional-programming.html">brief overview of Functional Programming</a>, this is the first in a series of blog posts describing how to make use of FP principles in C#, and hopefully make your projects more predictable, reliable, and maintainable.

<!--more-->

You may have thought that to reap the benefits of FP, you need to use a [purely functional language](https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Functional_languages) like Haskell or Elm. But that is not the case and its possible to make even object oriented languages such as C# more functional.

What's more, understanding the theory of FP concepts is not too difficult. In Robert C Martin's [*Clean Code*](https://www.amazon.co.uk/dp/0132350882/) (a book I'm guessing many developers have read), "Uncle Bob" advocates:
- Keeping functions small
- Not repeating code (the DRY principle)
- Only doing one thing
- Avoiding side effects, and
- Only having one level of abstraction

All of the above comes naturally when programming in a functional style, even though the examples in *Clean Code* are written in Java.

In other words, the difference between OO and FP boils down to how you compose your application rather than the features of the language or framework.


## Use Expressions and Reduce Side Effects

With regards to C# (and probably other languages), an expression is something that returns a value, whereas a statement does not. This means that expressions can be chained together. Statements are often executed for a side effect, for example, a method that returns `void` but changes the value of public or private members.

However, FP is naturally expression-based; almost everything produces some result.

To illustrate, let’s look at the following code snippets.

```c#
public class Program
{
    private static string aMember = "StringOne";

    //HyphenatedConcat is a statement that produces a side effect (appending to aMember)
    public static void HyphenatedConcat(string appendStr)
    {
        aMember += '-' + appendStr;
    }

    public static void Main()
    {
        HyphenatedConcat("StringTwo");
        Console.WriteLine(aMember);
    }
}
```
This will produce the following output: `StringOne-StringTwo`.

The next version implements `HyphenatedConcat` as a pure function. Recall that a pure function:
- Has no side effects. The function doesn't change any variables or the data of any type outside of the function.
- Is consistent and predictable. Given the same set of input data, it will always return the same output value.

```c#
public class Program
{
    public static string HyphenatedConcat(string s, string appendStr)
    {
        return (s + '-' + appendStr);
    }

    public static void Main(string[] args)
    {
        string s1 = "StringOne";
        string s2 = HyphenatedConcat(s1, "StringTwo");
        Console.WriteLine(s2);
    }
}
```

This second version produces the same output but in a functional style. Note that because FP variables should be immutable, the returned concatenated value is stored in another variable, `s2`.

> One approach that can be very useful is to write functions that are locally impure (that is, they declare and modify local variables) but are globally pure. Such functions have many of the desirable composability characteristics, but avoid some of the more convoluted functional programming idioms, such as having to use recursion when a simple loop would accomplish the same thing.[^1]

You may be thinking that there isn't really much difference between the two styles, but don't underestimate the benefit of pure functions, especially when we start chaining these functions together. I'll revisit method chaining in a future post, but as a teaser have a look at these code snippets[^2]:

```c#
// Imperative style
var orderId = Guid.Parse("9043f30c-446f-421f-af70-234fe8f57c0d");
var orderBL = new OrderBL();
orderBL.InitializeOrder(orderId); // None of these methods have a return value, but they do alter class members
orderBL.ValidateOrder(orderId);
orderBL.ProcessOrder();
orderBL.SaveOrder();
```

```c#
// Functional style
var orderId = Guid.Parse("9043f30c-446f-421f-af70-234fe8f57c0d");
var orderBL = new OrderBL();
orderBL.InitializeOrder(orderId) // All of these methods return a new instance of OrderBL
       .ValidateOrder(orderId)
       .ProcessOrder()
       .SaveOrder();             // except SaveOrder()
```

`SaveOrder` does not have a return value and thus must be the last method of the chain. Writing to a database or log is by definition a side effect, which is why I've named this section "*reduce* side effects" and not "*eliminate* side effects".

## First Class Functions

When a language treats functions as a data type, the functions are said to be first class citizens of that language.

This enables higher-order functions, which are functions that accept other functions as parameters and/or returns other functions. Higher-order functions allow us to compose functionality not through complex inheritance structures, but by substituting behaviour according to the function signature.

C# does this through LINQ, namely: generics, extension methods, delegation, and lambda expressions.

An extension method is actually a higher-order function which add functionality such as sorting, filtering, or transforming the values of any type that inherit from `IEnumerable<T>`.

As an example, let's look at some code that filters a list for just the evenly numbered elements and then sorts them:

```c#
// Imperative style
var idx = 0;
while (idx < myList.Count)
{
    if (myList[idx] % 2 != 0)
        myList.RemoveAt(idx);
    else
        ++idx;
}

myList.Sort();
```

Here, we have a while loop which indicates there is some mutable state because in order for the loop to run, the exit condition must initially be false, and then some side effect within that loop must change the condition so that the loop may terminate.

From the code we can see that both the index and the list change. Mutating `idx` isn't too much of a problem since it is likely isolated to this particular block, but changing the list could have system-wide implications if it's visible outside of the method where this code is located.

Next, we have `myList.Sort()`. This is a void method, and as it doesn't return anything we must be invoking it solely for its side effect. In other words, although `Sort` does indeed sort the list, it does so by mutating the list.

Here is the same code written using LINQ:

```c#
// Functional approach
myList.Where(x => x % 2 == 0) // We tell the higher-order functions how to handle each element 
      .OrderBy(x => x);       // in the sequence and wait for the return values.
```

This declarative approach has abstracted away much of the plumbing code we saw in the imperative approach. As a result, we can focus on solving the problem, rather than having to mentally parse a bunch of code that's really only there to satisfy the compiler.

Also, what might not be immediately obvious is that *`myList` is never changed*. Rather than mutating the original list, the LINQ extension methods create new sequences, where each item in the input sequence is mapped, or not mapped, to a resulting sequence depending on the corresponding lambda expression. As such, we can safely reference `myList` from elsewhere if necessary[^3].

[^1]: {% include citation.html key="refactor-to-pure-funcs" %}
[^2]: {% include citation.html key="howto-fluent-api" %}
[^3]: {% include citation.html key="functions-as-data" %}
