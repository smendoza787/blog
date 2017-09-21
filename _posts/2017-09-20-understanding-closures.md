---
published: true
layout: post
title: "Understanding Closures in JavaScript (With Delicious Mexican Food)"
author: "Sergio"
permalink: /2017/09/20/understanding-closures
date: '2017-09-20 16:16:01 -0600'
---

Closures are something we are sure to encounter everyday in our JavaScript programs. So I wanted to come up with a simple and practical example to get an idea of what a closure is, and how you would use it.

# What is a closure?

Here are a few different definitions of closure I came across, to really drill into your head what a closure is defined as:

- A closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

- A closure is the combination of a function and the lexical environment within which that function was authored.

- A closure is an inner function that has access to the outer functions variables.

A closure has access to 3 different scopes; its own scope and variables, its enclosing function's scope and variables, and the global scope and its variables. The closure also has access to the outer function's argument parameters as well.

# How to create a closure

Let's create a simple example of a closure. Let's say you're at a delicious Mexican restaurant, and we need a function that will return another function with access to the outer functions scope

```javascript
function takeOrder(mexicanDish) {
  var orderCall = "Here is your " + mexicanDish + " with ";

  function insertIngredient(mainIngredient) {
    console.log(orderCall + mainIngredient);
  }

  return insertIngredient;
}

var steakBurrito = takeOrder('burrito');

steakBurrito('steak'); // "Here is your burrito with steak"
```

When we first call our `takeOrder()` function, we are passing it an argument of the type of mexican food we want. This can be anything they offer; a taco, burrito, quesadilla.... you get the idea.

The point is, when we call `takeOrder('burrito')`, it returns to us a function, `insertIngredient()`. We set this equal to a variable, that we will invoke later, named `steakBurrito`, because that's what our final order will look like. So you can imagine that our variable, `steakBurrito`, is really just the `insertIngredient()` function in disguise, waiting to be called with a mainIngredient.

Before we call that function, we can observe that the inner function, `insertIngredient()`, is holding on to a variable `orderCall` that was written in our outer function, `takeOrder()`.

We finally call `steakBurrito()` and pass in the main ingredient of `'steak'`, which results in our function logging out our final `orderCall`.

In some languages, this would have returned an error, because we technically already invoked and ran `takeOrder()` when we assigned it to our variable and it shouldn't be available anymore because the function finished its execution. But thanks to the magic of closures in JavaScript, our little inner function is still holding on for dear life to the variable in the outer function. And that's essentially what a closure is!

For clarity, and because I'm still pretty hungry, we can also order a chicken taco!

```javascript
var chickenTaco = takeOrder('taco');
chickenTaco('chicken'); // "Here is your taco with chicken"
```

There are a ton of practical uses for closures, and more complex ways to implement them. This was just a simple example that helped me identify what a closure is at the basic level. Hopefully this can push someone over the hump, and into the a-ha moment of understanding closures in JavaScript!
