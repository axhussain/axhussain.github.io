---
layout: post
title: Using Functional Programming Principles in C# (Part 2)
tags: [functional-programming, c#, immutability]
comments: true
excerpt_separator: <!--more-->
---

In my <a href="/2020/12/24/using-fp-in-csharp-pt1.html">last post</a> I mentioned we should try to reduce side-effects in our functions. One of the easiest ways to do this is to enforce immutability.

<!--more-->

## Immutability

When I first read about Functional Programming, I was immediately struck by the oddness of immutability. I remember thinking, surely variables that can’t change are just constants – how can an application even work with just constants?! But at the time I had yet to understand pure and higher-order functions.

Finally the penny dropped: functions take an input and map it to the output. If the output is an object, it will be a *new* instance of that object, even if the input parameter and returned object are of the same type. *The input value never changes.*

The problem with mutable variables becomes most apparent in multi-threaded or asynchronous applications. For example, imagine our application uses the following `DateRange` class:


```c#
public class DateRange
{
    public DateTime Start { get; set; }
    public DateTime End { get; set; }

    public bool DateIsInRange(DateTime checkDate)
    {
        // Less than zero - The instance is earlier than checkDate
        // Zero - The instance is the same as checkDate
        // Greater than zero - The instance is later than checkDate
        return Start.CompareTo(checkDate) <= 0 && End.CompareTo(checkDate) >= 0;
    }
}
```

Which gets instantiated and called like this:

```c#
var reportDateRange = new DateRange { Start = DateTime.Parse("2020-12-01"), End = DateTime.Parse("2020-12-31") };

var transactionDate = DateTime.Parse("2020-12-29");

if (reportDateRange.DateIsInRange(transactionDate)) {
    // Do something
}
```

If either of `reportDateRange`'s public properties get amended by another thread, then `reportDateRange.DateIsInRange(transactionDate)` could be `true` on the first call but `false` on subsequent calls.

This is a code-smell to Functional Programmers because pure functions should *always* return the same result for the same input.

To ensure that another thread can’t change the values we could lock the object. But the preferred solution is to make the object immutable – there’s no need to implement a locking strategy if the object can’t change[^1].

C# 6 introduced getter-only auto-properties which we can use in the `DateRange` class:

```c#
public class DateRange
{
    public DateTime Start { get; }
    public DateTime End { get; }

    public DateRange(DateTime start, DateTime end)
    {
        Start = start;
        End = end;
    }

    public bool DateIsInRange(DateTime checkDate)
    {
        return Start.CompareTo(checkDate) <= 0 && End.CompareTo(checkDate) >= 0;
    }
}
```

`Start` and `End` now have to be set in a constructor and cannot be amended once the object has been constructed (it’s the equivalent to setting the properties as `readonly`).

To amend the date range, you’ll need to create a new instance of `DateRange`:

```c#
// Note, I *have* to pass the dates in the Constructor and can’t use the object initializer syntax
var reportDateRange = new DateRange(DateTime.Parse("2020-12-01"), DateTime.Parse("2020-12-31"));

var newDateRange = new DateRange(reportDateRange.Start, DateTime.Parse("2021-01-31"));
```

The compiler will not allow something like `reportDateRange.End = DateTime.Parse("2021-01-31");`.


[^1]: {% include citation.html key="functions-as-data" %}
