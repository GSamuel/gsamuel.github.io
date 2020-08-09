---
layout: post
title:  "Double negatives should not not be avoided"
summary: Why positive form first improves your boolean logic.
author: Gideon Hoogeveen
date: '2020-08-08 22:47:23 +0200'
category: boolean
thumbnail: /assets/img/posts/boolean-true-or-false.jpg
keywords: kotlin, boolean, true, false, double negation, double negatives
permalink: /blog/double-negations
---

Booleans are the bread and butter of decision making in most modern programming languages and in Kotlin this is no exception.
We will discuss some ways to clean up your boolean logic and explore some best practices in giving good names to our binary friends.

## Double negations are not not bad.
Beware of code that uses double negations. Double negations are hard to read and understand. Double negations significantly increase the amount of bugs and errors that can potentially hide in your code. Try the following exercise to better understand the problem.

> Which number will be printed to the console given that the user is typing?

{% highlight kotlin %}
// Double negation
val isNotTyping = false

fun doSomethingA() {
  if(!isNotTyping)
    println(1)
  else
    println(2)
}
{% endhighlight kotlin %}

{% highlight kotlin %}
// Single negation
val isNotTyping = false

fun doSomethingB() {
  if(isNotTyping)
    println(3)
  else
    println(4)
}
{% endhighlight kotlin %}

{% highlight kotlin %}
// No negation
val isTyping = true

fun doSomethingC() {
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

If you really need the negative form of a method consider providing the positive part first and let the negative form call the positive form. Most of the time it is easier to implement the positive form anyway. Letting the negative form call the positive form has the added benefit of avoiding duplication and therefore limits the amount of code that has to be touched when requirements change.

{% highlight kotlin %}
fun isNotDone():Boolean = !isDone()

fun isDone():Boolean = progress == 100
{% endhighlight kotlin %}

## Clean up existing code

A lot of times you have to deal with code that has already been written by you or an other person. As per the Boy Scout Rule we should Leave our code better than we found it.

Let's say you found this piece of code that deals with boxes containing integers.

> Which steps would you take to improve this code?

You can use the Kotlin [Playground][playground] to test the code in your browser.

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(!isNotEmpty())
    	println("box is empty")
    else
    	println("box contains: ${value}")
}

class Box(val value:Int?) {
    fun isNotEmpty() = value != null
}
{% endhighlight kotlin %}

Make sure you check if your code still compiles and behaves the same way after each step.

Step 1: Introduce positive form.

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(!isNotEmpty())
    	println("box is empty")
    else
    	println("box contains: ${value}")
}

class Box(val value:Int?) {
    fun isNotEmpty() = value != null
    fun isEmpty() = !isNotEmpty() // Introduce positive form which calls the other form.
}
{% endhighlight kotlin %}

Step 2: Replace each double negative with a call to the new function.

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(isEmpty()) //Replace all double negatives
    	println("box is empty")
    else
    	println("box contains: ${value}")
}

class Box(val value:Int?) {
    fun isNotEmpty() = value != null
    fun isEmpty() = !isNotEmpty()
}
{% endhighlight kotlin %}

Step 3:
If negative form is only used in the positive form use the [Inline Function][inline-function] Refactor method to move to merge the body of the negative form (isNotEmpty()) to the positive form (isEmpty()).

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(isEmpty())
    	println("box is empty")
    else
    	println("box contains: ${value}")
}

class Box(val value:Int?) {
    //Remove unused isNotEmpty() method. If you still need it make sure it calls isEmpty()
    fun isEmpty() = !(value != null) // replace method call by implementation.
}
{% endhighlight kotlin %}

Step 4:
Clean up the positive form a little further

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(isEmpty())
    	println("box is empty")
    else
    	println("box contains: ${value}")
}

class Box(val value:Int?) {
    fun isEmpty() = value == null // making method implementation more expressive
}
{% endhighlight kotlin %}

If you don't want your logic to be around whether the box is empty you can still introduce a method using a positive form to see whether the box has a value.

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(hasValue()) //Replaced isEmpty() with hasValue()
    	println("box contains: ${value}") // Swap
    else
    	println("box is empty") // Swap
}

class Box(val value:Int?) {
    fun hasValue() = !isEmpty() // Introduce method with positive form
    fun isEmpty() = value == null
}
{% endhighlight kotlin %}

There's no reason to use complex and confusing double negatives in your code. So do yourself us all a favor. **Don't not avoid double negations**.


[inline-function]: https://refactoring.com/catalog/inlineFunction.html
[playground]: https://play.kotlinlang.org/