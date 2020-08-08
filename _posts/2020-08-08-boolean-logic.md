---
layout: post
title:  "Double negations are not not hard"
summary: Why positive form first improves your boolean logic.
author: Gideon Hoogeveen
date: '2020-08-08 22:47:23 +0200'
category: boolean
thumbnail: /assets/img/posts/boolean-true-or-false.jpg
keywords: kotlin, boolean, true, false, double negation, 
permalink: /blog/double-negations
---

Booleans are the bread and butter and decision making in most modern programming languages and in Kotlin this is no exception.
We will discuss some ways to clean up your boolean logic and explore some best practices in giving good names to our binary friends.

## Double negation are not not bad.
Beware of code that uses double negations. Double negations are harder to read and understand then their positive counterparts. Double negations significantly increase the amount of bugs and errors that can potential hide in your code. Try the following exercise to better understand the problem with double negatives.

> Which number will be printed to the console given that the user is typing?

{% highlight kotlin %}
// Double negation
fun doSomethingA(val isNotTyping:Boolean) {
  if(!isNotTyping)
    println(1)
  else
    println(2)
}
{% endhighlight kotlin %}

{% highlight kotlin %}
// Single negation
fun doSomethingB(val isNotTyping:Boolean) {
  if(isNotTyping)
    println(3)
  else
    println(4)
}
{% endhighlight kotlin %}

{% highlight kotlin %}
// No negation
fun doSomethingC(val isTyping:Boolean) {
  if(isTyping)
    println(5)
  else
    println(6)
}
{% endhighlight kotlin %}

Given that the user is typing calling these methods will result in 1, 4 and 5.

In my experience most programmers take longer and make more mistakes reading the code that uses double negations compared to the code where less negations are used. This is why you should always strive to write boolean variables and parameters in their positive form.

The same rule should be applied to methods which return a boolean.

{% highlight kotlin %}
fun isNotOdd(value:Int):Boolean {
  // Implementation
}
{% endhighlight kotlin %}

Will be less clear and should be replaced with:

{% highlight kotlin %}
fun isEven(value:Int):Boolean {
  // Implementation
}
{% endhighlight kotlin %}

## Try to start positive

If you really need the negative form of a method consider providing the positive part first and let the negative form call the positive form. Most of the time it is easier to implement the positive form anyway. Letting the negative form call the other positive form also has the added benefit of avoiding duplication and therefore limits the amount of code that has to be touched when requirements change.

{% highlight kotlin %}
fun isNotDone():Boolean = !isDone()

fun isDone():Boolean = progress == 100
{% endhighlight kotlin %}