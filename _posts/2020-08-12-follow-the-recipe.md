---
layout: post
title:  "Follow The Recipe"
summary: Reduce nesting by using early returns
author: Gideon Hoogeveen
date: '2020-08-12 22:25:07 +0200'
category: kotlin
thumbnail: /assets/img/posts/nesting.jpg
keywords: reduce nesting, recipe, complexity, refactor, early return
permalink: /blog/follow-the-recipe
preview: true
---

## You like turtles?
Today is your best friends birthday. Your friend is in love with turtles, so you decided to give one as a present. The problem is you have never seen a real turtle before. The only thing you know about turtles is that they can swim, have four legs and are reptiles. As you walk into the shop you see a lot of different species of animals. You decide to write a simple program to determine whether an animal is a turtle.

The code that you come up with looks like this:

{% highlight kotlin %}
fun isTurtle() {
    var isTurle:Boolean = false
    if(canSwim()) {
        if(hasFourLegs()) {
            if(isReptile()) {
                isTurtle = true
            } else {
                isTurtle = false
            }
        } else {
            isTurtle = false
        }
    } else {
        isTurle = false
    }

    return isTurtle
}
{% endhighlight kotlin %}

## Maintenance nightmare

After running your code against all animals in the shop you still have a problem. There are still multiple animals that can swim, have four legs and are reptiles.

You try to add more conditions to your program, but because there are already so many conditions it becomes much harder to add more without making a mistakes. The horror if your present happens to be a crocodile and the quests will be eaten instead of the birthdaycake.

> What would you change to make this program more readable?

## Refactor to a recipe

The first thing you realize is that when an animal cannot swim it cannot be a turtle. Therefore we can safely take out te **canSwim()** condition and use an early return when the animal cannot swim.

{% highlight kotlin %}
fun isTurtle() {

    // Check whether current animal can swim or not.
    if(!canSwim()) 
        return false // If the animal can't swim, we are done.

    // From now on we know that we are dealing with an animal that can swim

    var isTurle:Boolean = false

    // Therefore we don't have to check whether the turtle can swim anymore, so we remove the check.

    if(hasFourLegs()) {
        if(isReptile()) {
            isTurtle = true
        } else {
            isTurtle = false
        }
    } else {
        isTurtle = false
    }

    return isTurtle
}
{% endhighlight kotlin %}

After you have rewritten the checks on **canSwim()** it becomes clear you can do the same with the check on **hasFourLegs()** aswell.

{% highlight kotlin %}
fun isTurtle() {

    if(!canSwim())
        return false

    if(!hasFourLegs())
        return false

    var isTurle:Boolean = false

    if(isReptile()) {
        isTurtle = true
    } else {
        isTurtle = false
    }

    return isTurtle
}
{% endhighlight kotlin %}

Eventhough the method is already becoming way more readable, you see no reason to stop now.

{% highlight kotlin %}
fun isTurtle() {

    if(!canSwim())
        return false

    if(!hasFourLegs())
        return false

    if(!isReptile())
        return false
    
    // When this part is reached, we know our animal can swim, has four legs and is a reptile and therefore  (probably) is a turtle.

    return true
}
{% endhighlight kotlin %}

## Adding new requirements

Finally to distinguish the turtle from the crocodile you come up with one final check. Does the animal have a shell?
Now that you have written your code like a recipe it becomes trivial for you to add this new condition. A recipe can easily be read from top to bottom.

{% highlight kotlin %}
fun isTurtle() {

    if(!canSwim())
        return false

    if(!hasFourLegs())
        return false

    if(!isReptile())
        return false

    // Distinguish between turtles en crocodiles
    if(!hasShell())
        return false

    return true
}
{% endhighlight kotlin %}


## But, aren't early returns a bad thing?
Some would argue that early returns make your code harder to understand because there are multiple places where you method could end. In my experience this is never a problem until your methods become very long and complex. When this happens you should probably deal with those long methods instead i.e. by reducing their length and complexity first.

## The Recipe
The general recipe is as follows:
1. Try to do X
2. If X didn't work -> stop
3. continue with the assumption that X worked

In the following example you can see this pattern at work. The fridge in this example is a simple [state machine][state-machine] and all illegal transitions will throw an exception.

{% highlight kotlin %}
fun main() {
    val fridge = Fridge()
    fridge.open()
    fridge.putElephantInFridge()
    fridge.close()

    fridge.open()
    fridge.takeElephantFromFridge()
    fridge.close()
}

class Fridge {

    var isOpen: Boolean = false
    var isEmpty: Boolean = true

    fun open() {
        if(isOpen)
            throw IllegalStateException("Unable to open an already opened fridge")

        isOpen = true
        println("Fridge opened")
    }

    fun close() {
        if(!isOpen)
            throw IllegalStateException("Unable to close an already closed fridge")

        isOpen = false
        println("Fridge closed")
    }

    fun putElephantInFridge() {
        if(!isOpen)
            throw IllegalStateException("Unable to put elephant in a closed fridge")

        if(!isEmpty)
            throw IllegalStateException("Unable to put elephant in fridge because there is no room left")

        isEmpty = false
        println("Elephant put in fridge")
    }

    fun takeElephantFromFridge() {
        if(!isOpen)
            throw IllegalStateException("Unable to take an elephant from a close fridge")

        if(isEmpty)
            throw IllegalStateException("There is no elephant to take out of the fridge")

        isEmpty = true
        println("Elephant taken from fridge")
    }
}
{% endhighlight kotlin %}

## Assertions
To make this code cleaner you can use Kotlins **require(value:Boolean)** and **check(value:Boolen)** methods to assert that a certain condition has been met. Now your code becomes even more readable and elegant.

{% highlight kotlin %}
fun main() {
    val fridge = Fridge()
    fridge.open()
    fridge.putElephantInFridge()
    fridge.close()

    fridge.open()
    fridge.takeElephantFromFridge()
    fridge.close()
}

class Fridge {

    var isOpen: Boolean = false
    var isEmpty: Boolean = true

    fun open() {
        check(isOpen) { "Unable to open an already opened fridge" }

        isOpen = true
        println("Fridge opened")
    }

    fun close() {
        check(!isOpen) {"Unable to close an already closed fridge"}

        isOpen = false
        println("Fridge closed")
    }

    fun putElephantInFridge() {
        check(!isOpen) { "Unable to put elephant in a closed fridge" }

        check(!isEmpty) {"Unable to put elephant in fridge because there is no room left"}

        isEmpty = false
        println("Elephant put in fridge")
    }

    fun takeElephantFromFridge() {
        check(!isOpen) { "Unable to take an elephant from a close fridge" }

        check(isEmpty) { "There is no elephant to take out of the fridge" }

        isEmpty = true
        println("Elephant taken from fridge")
    }
}
{% endhighlight kotlin %}

[state-machine]: https://en.wikipedia.org/wiki/Finite-state_machine