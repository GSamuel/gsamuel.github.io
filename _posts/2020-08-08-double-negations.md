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
I will discuss with you some ways to clean up your boolean logic and you will explore some best practices in giving good positive names to our binary friends.

## Double negations are not not bad.
You should not use double negations in your code because they are hard to read and understand. They also significantly increase the amount of bugs and errors that can hide in your program. I invite you to try the following exercise to better understand why Double negatives form a problem.

> Which numbers will be printed to the console?

{% highlight kotlin %}
val isNotTyping = false
val isTyping = true

fun main() {
    doubleNegation()
    singleNegation()
    noNegation()
}

fun doubleNegation() {
  if(!isNotTyping)
    println(1)
  else
    println(2)
}

fun singleNegation() {
  if(isNotTyping)
    println(3)
  else
    println(4)
}

fun noNegation() {
  if(isTyping)
    println(5)
  else
    println(6)
}
{% endhighlight kotlin %}

Most programmers take longer and make more mistakes interpreting the code that uses double negations compared to the code where less negations are used. This is why you should always try to write boolean variables and parameters in their positive form.

Here is the solution. Did you get it right?

{% highlight kotlin %}
1
4
5
{% endhighlight kotlin %}

Parameters and variables are not the only places where problematic usage of double negations can be found. Methods, that have a boolean as return type, suffer from the same problems.

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

If you really need the negative form of a method consider providing the positive part first and let the negative form call the positive form. Most of the time it is easier to implement the positive form anyway. Letting the negative form call the positive form has the added benefit of avoiding duplication, which limits the amount of code that has to be touched when requirements change.

{% highlight kotlin %}
fun isNotDone():Boolean = !isDone()

fun isDone():Boolean = progress == 100
{% endhighlight kotlin %}

## Clean up existing code

Often you have to deal with code that has already been written by you or an other person. As per the Boy Scout Rule we should Leave our code better than we found it.

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

Instead of using a method named **isNotEmpty** to see whether a box is full it would probably make more sense to introduce a method called **isFull**. Often you have to search a little longer for names that better convey the right meaning.

{% highlight kotlin %}
fun main() {
    Box(7).print()
    Box(null).print()
}

private fun Box.print() {
    if(isFull()) //Replaced isEmpty() with isFull()
    	println("box contains: ${value}") // Swap
    else
    	println("box is empty") // Swap
}

class Box(val value:Int?) {
    fun isFull() = !isEmpty() // Introduced new method in its positive form
    fun isEmpty() = value == null
}
{% endhighlight kotlin %}

There's no reason to use complex and confusing double negatives in your code. So do us all a favor. **Don't not avoid double negations**.


[inline-function]: https://refactoring.com/catalog/inlineFunction.html
[playground]: https://play.kotlinlang.org/