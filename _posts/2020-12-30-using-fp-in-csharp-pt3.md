---
layout: post
title: Use Expressions instead of Statements
tags: [functional-programming, c#, expressions, operators]
comments: true
excerpt_separator: <!--more-->
---

This is the 3rd part in my series of applying Functional Programming principles in C#.

Functional Programming is naturally expression-based; almost everything produces a result. With regards to C#, although many of its programming constructs are statement based, it does provide a number of operators we can use instead of statement-based alternatives.

<!--more-->

## Null-conditional operators (`?.` and `?[]`)

The `?.` operator allows you to access members *only* when the parent type is not-null, returning a null result otherwise, e.g:

```c#
int? length = people?.Length;  //length will be null if people is null
```

The above is equivalent to:

```c#
int? length;

if (people != null) {
    length = people.Length;
}
```

or, we could even use the ternary conditional operator (`?:`), but I think you'll agree that in this case `?.` looks cleaner:

```c#
int? length = (people != null) ? (int?)people.Length : null;
```

Null-conditionals also works on lists and arrays, using the `?[]` syntax:

```c#
Type personType = people?[0].GetType();  //personType will be null if people is null
```


## Null-coalescing operator (`??`)

The `??` null-coalescing operator provides a default value when the outcome is null. For example:

```c#
var result = a ?? b;
```

The operation evaluates `b` *only* if `a` is null. (`a` must be a nullable or reference type)


## Expression Bodied Members

Whenever the logic for a supported member (such as a method or property) consists of a single expression, we can rewrite our code to use a more concise and readable syntax: `member => expression;`

For example, take a look at this simple Singleton implementation:

```c#
public sealed class MySingleton
{
    private static MySingleton _instance;

    private MySingleton()
    {
    }
 
    public static MySingleton Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new MySingleton();
            }
            return _instance;
        }
    }
    // Other stuff here
}
```

The entire getter can be reduced to a single expression:

```c#
public static MySingleton Instance
{
    get
    {
        return _instance ?? (_instance = new MySingleton());
    }
}
```

So now we have a single expression we can take advantage of expression-bodied member syntax:

```c#
public static MySingleton Instance => _instance ?? (_instance = new MySingleton());
```

See how much cleaner than looks?

Expression-bodied members were first introduced in C# 6 with only methods and properties. But with C# 7, several new members have been included, namely[^1]:

- Methods
- Properties
- Constructor
- Destructor
- Getters
- Setters
- Indexers

In my next post I'll take a look at replacing a statement with an expression when there is no special operator that we can use.

[^1]: {% include citation.html key="expr-bodied-membrs" %}
