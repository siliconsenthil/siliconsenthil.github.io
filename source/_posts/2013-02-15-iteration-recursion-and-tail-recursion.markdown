---
layout: post
title: "Iteration, Recursion and Tail recursion"
date: 2013-02-15 12:38
comments: true
categories: [scala, programming]
keywords: scala,programming,recursion
description: "Understanding the differences between Iteraion, Recursion and Tail Recursion using a Scala example."
---

**Problem:** Given _a_ and _b_,<br/>find the sum of range of numbers between _a_ and _b_.<br>E.g. Given 1 and 4, the result is 10 (1+2+3+4). Simple.


**Language:** Scala

Let's do this with programmers' favorite, Iteration
<!--more-->

```scala
    def iterative_sum(a: Long, b: Long):Long =  {
      var result = 0L;
      for(i <- a until b){result=result+i}
      return result+b;
    }

```

If we had to do this with recursion,

```scala
def sum(a: Long, b: Long): Long = if(a > b) 0 else a + sum(a+1, b)
```

This works for smaller values. There's a problem with this for bigger values because the way it's implemented. If we see how it works, it would be obvious to figure that out. Assume the red grid as a limited size stack.

{% img http://i50.tinypic.com/1218htk.png 410px %}

It keeps putting the values to stack till it gets it could evaluate to a value. Till then the expressions are kept in stack.
Now, you can it throws _StackOverflowError_ for larger input.

{% img http://i45.tinypic.com/2mw8ry1.png 410px %} 

This is the problem with recursion. This might be one reason programmers tend to avoid recursion as it might not be obvious to guess when it fails.

If we could write a recursion function that does not force to use stack then we can avoid this error. And that is called as tail recursion. The way is that the last line of the recursion function should be just the recursion and not somthing else to do with the result.

With the help of nested function construct in scala, we could write something like this. Here <code>loop(a,0)</code> is the recusion call. As you can see this is the only expression, it does not require stack to store intermediate result.

```scala
    def tail_sum(a: Long, b: Long): Long ={
      def loop(a: Long, acc: Long): Long = {
        if(a > b) acc
        else loop(a+1, acc+a)
      }
      loop(a, 0)
    } 
```

I tried the above all in for a large value (>6998 in my mac) saw recursion failing while iteration and tail recursion are able to return result.

{% gist 4959395 IterationRecursionTailRecursion.scala %}

