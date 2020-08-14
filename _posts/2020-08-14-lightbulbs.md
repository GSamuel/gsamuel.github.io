---
layout: post
title:  "Lightbulbs"
summary: Have you tried turning it off and on again?
author: Gideon Hoogeveen
date: '2020-08-13 21:40:07 +0200'
category: [kotlin, refactoring]
thumbnail: /assets/img/posts/recipe.jpg
keywords: 
permalink: /blog/lightbulbs
preview: true
---

## Lightbulbs

When we press the button the light turns on.

{% highlight kotlin %}
fun main() {
    pressButton()
}

fun pressButton() {
    println("Button Pressed")
    turnOnLight()
}

fun turnOnLight() {
    println("Light turned on")
}
{% endhighlight kotlin %}

When we press the button again the light turns off.

{% highlight kotlin %}

var lightOn: Boolean = false

fun main() {
    pressButton()
    pressButton()
}

fun pressButton() {
    println("Button Pressed")
    if(lightOn)
        turnLightOff()
    else
        turnLightOn()
}

fun turnLightOn() {
    lightOn = true
    println("Light turned on")
}

fun turnLightOff() {
    lightOn = false
    println("Light turned off")
}
{% endhighlight kotlin %}

## More requirements
The only constant in software development is Change and to rub it in our face, Your Manager comes in with a new set of requirements.
* Instead of 1 button, I want multiple buttons. These buttons could toggle the same light on or off or they could toggle a different light. A button only changes the state of a single lightbulb.

## Our model can't handle these changes :O
These new requirements cause a lot of problems for or existing code. We made the assumption that there will only ever be one button and only one Light. Just adding new methods and adding more state won't solve our problem, because it will make our code very complex very fast.

## Refactor first then add the new features.
Refactoring is the art of changing the way your code is organized without changing it's behaviour. In the past I did make the mistake of refactoring and incorporating new features at the same time.

{% highlight kotlin %}

val light = Light()

fun main() {
    pressButton()
    pressButton()
}

fun pressButton() {
    println("Button Pressed")
    if(light.on)
        light.turnOff()
    else
        light.turnOn()
}

class Light {
    var on:Boolean = false
    fun turnOn() { 
        check(!on) { "Light is already on!" }
        on = true
        println("Light turned on")
    }

    fun turnOff() {
        check(on) { "Light is already off!" }
        on = false
        println("Light turned off")
    }
}

{% endhighlight kotlin %}

When we run our code. It should behace exactly the same as before.

Now let's extract our button to it's own model as well.

{% highlight kotlin %}

fun main() {
    val light = Light()
    val button = Button(light)

    button.press()
    button.press()
}

// A button operates on a light, so it needs a reference to the light.
class Button(val light:Light) {
    fun press() {
        println("Button Pressed")
        if(light.on)
            light.turnOff()
        else
            light.turnOn()
    }
}

class Light {
    var on:Boolean = false
    fun turnOn() { 
        check(!on) { "Light is already on!" }
        on = true
        println("Light turned on")
    }

    fun turnOff() {
        check(on) { "Light is already off!" }
        on = false
        println("Light turned off")
    }
}

{% endhighlight kotlin %}

## Code Smell
Currenntly our button is responsible for turning the light on and off. The Button first asks a question to the light: Are you on or off? Depending on the lights answer the button will tell the light to witch state to switch. If the button makes a mistake, the light will throw an exception and tell the button It can't do as requested.

When your code is asking a lot of questions to make a decision This can be an indicator of a design problem.

> What would you change to make the interaction between the button and the light simpeler?

## Classes are both data and behaviour
Data and behaviour that acts upon that data are closely coupled and should reside close together. 

By making the light responsible for and its state transitions we get a Light that is better.

{% highlight kotlin %}
fun main() {
    val light = Light()
    val button = Button(light)

    button.press()
    button.press()
}

// A button operates on a light, so it needs a reference to the light.
class Button(val light:Light) {
    fun press() {
        println("Button Pressed")
        light.toggle()
    }
}

class Light {
    var on:Boolean = false

    fun toggle() {
        if(on)
            turnOff()
        else
            turnOn()
    }

    private fun turnOn() { 
        check(!on) { "Light is already on!" }
        on = true
        println("Light turned on")
    }

    private fun turnOff() {
        check(on) { "Light is already off!" }
        on = false
        println("Light turned off")
    }
}
{% endhighlight kotlin %}

Because we don't want **turnOn()** and **turnOff()** be directly available as methods on our light anymore, we can make them private. We can also reduce the complexity of the **toggle()** method and removing the **turnOn()** and **turnOff()** methods completely. If in the future there is a need to reintroduce these methods you can write them again, but for now they only add unnecessary complexity. If you are scared to throw code away remember that you always have version control. Also see [YAGNI][yagni].

{% highlight kotlin %}
fun main() {
    val light = Light()
    val button = Button(light)

    button.press()
    button.press()
}

// A button operates on a light, so it needs a reference to the light.
class Button(val light:Light) {
    fun press() {
        println("Button Pressed")
        light.toggle()
    }
}

class Light {
    var on:Boolean = false

    fun toggle() {
        if(on)
            println("Light turned off")
        else
            println("Light turned on")

        on = !on
    }
}
{% endhighlight kotlin %}


[playground]: https://play.kotlinlang.org/
[yagni]: https://enterprisecraftsmanship.com/posts/yagni-revisited/