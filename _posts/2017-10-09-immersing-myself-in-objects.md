---
published: true
layout: post
title: "Immersing Myself in JavaScript Objects"
author: "Sergio"
permalink: /2017/10/09/immersing-myself-in-objects
date: '2017-10-09 16:16:01 -0600'
---

There are 6 data types in JavaScript:
  
  - Object
  - Number
  - String
  - Boolean
  - Null
  - Undefined

You can split this group of data types into 2 groups: complex and primitive. Primitive data types are _immutable_ (unchangeable), while Complex data types are _mutable_ (so changeable, it's not even funny).

All data types besides Objects are considered primitive data types. Today I want to focus on the most fundamental, most used and only complex data type in JavaScript; the Object.

<p align="center">
  <img height="300" src="https://media.giphy.com/media/MvovQGsMBY9H2/giphy.gif" alt="objects">
</p>

# What is an Object?

At a fundamental level, you can think of an object in JavaScript much like an object in real life.

In JavaScript, an object is defined as **an unordered list of primitive data types that are stored as a series of key-value pairs.**

Objects are usually used for storing data and/or creating custom methods.

A coffee mug, for example, is a real, tangible object that possesses many properties and characteristics such as its color, size, material, manufacture date, etc.

You could represent this as a JavaScript object like this:

```javascript
  var coffeeMug = {
    color: 'blue',
    size: 'medium',
    material: 'ceramic',
    manufactureDate: 20171009
  };

  console.log(coffeeMug.color); // 'blue'
```

In our program, the `coffeeMug` object represents, well, a coffee mug with the properties of `color`, `size`, `material` and `manufactureDate`.

Each item on the `coffeeMug` object is considered a `property` on the object. The values of these properties could be any data type, including functions or even other objects!

Any property on an object whose value is a function is referred to as a **method**.

For example, if we were making a hi-tech coffee mug that knew how to brew its own cup of coffee, we could define a method on the object like this:

```javascript
  var coffeeMug = {
    coffee: {
      type: 'ground',
      brand: 'folgers'
    },
    brewCoffee: function() {
      console.log('Now brewing...');
    }
  };
```

We can now call `coffeeMug.brewCoffee()` to execute the method from our `coffeeMug` object.

<p align="center">
  <img height="300" src="https://media.giphy.com/media/3oEjI1JmchoJMbIJYQ/giphy.gif" alt="coffee">
</p>

# Creating Objects

There are 2 common ways to create an object:

  1. Object Literals
  2. Object Constructors

## Object Literals

The easiest way to create an object is by using literal brackets when defining your object:

```javascript
  var books = {};

  var banana = {
    color: 'yellow',
    shape: 'slender, curved',
    sweetness: 7,
    howSweet: function() {
      console.log('MMMMM MMM DAS SWEET!');
    }
  };
```

As you can see, the `books` variable refers to an empty object, while the `banana` variable refers to an object complete with properties and methods already defined inside. That banana has some personality!

<p align="center">
  <img height="300" src="https://media.giphy.com/media/yAqdjThdDEMF2/giphy.gif" alt="banana">
</p>

## Object Constructors

Another way to create objects is by using the **Object Constructor** function. It's a function that is used for initializing new objects, and is generally used with the `new` keyword.

```javascript
  var apple = new Object();
  apple.color = 'Red';
  apple.shape = 'round';
  apple.sweetness = 8;
```

These are all effective ways of creating objects for small programs, but what happens when out data sets get bigger, and we want to work with a lot more types of fruits?

Even with object literals, we would have to define each new fruit object with its different properties every time we discovered a new type of fruit. Depending on how massive our program ends up being, this would be a huge hassle.

# Patterns

Some very smart people have worked on solutions for dealing with repetitive tasks involving objects, so they devised a set of _patterns_ for dealing with the creation of multiple objects.

<p align="center">
  <img height="300" src="https://media.giphy.com/media/3o85gd3noLuSkE4Lkc/giphy.gif" alt="pattern">
</p>

## Constructor Pattern

The constructor pattern requires you make your own constructor function for the Object that you want to create. This constructor function will serve as a 'blueprint' for all future objects instantiated with the `new` keyword, while passing in the appropriate values for the objects properties.

```javascript
  function Fruit(color, sweetness, name, nativeLand) {
    this.color = color;
    this.sweetness = sweetness;
    this.name = name;
    this.nativeLand = nativeLand;

    this.showName = function() {
      console.log("This is a " + this.name);
    }
  }

  var mango = new Fruit("yellow", 5, "Mango", ["South America", "Central America", "West Africa"]);

  mango.showName();
  // "This is a Mango"
```

Notice how we use the `this` keyword to refer to the calling object, `mango`. `this` is used inside a method to refer to the current object. We instantiate the new object by calling `new Fruit()` and passing in the appropriate arguments to fill out the newly created object's properties.

Now whenever a new fruit is made with the `Fruit()` constructor function and the right arguments, it will now have the same blueprint as every other `Fruit` created with that function.

## Prototype Pattern

There is no Class hierarchy in JavaScript like there is in Java or C++, there are only objects. In a class-based language, you have the luxury of creating **Classes**, that will instantiate **Objects** while having those Objects inherit all of the methods and variables from its parent Class.

Javascript is **NOT** a class-based language, rather, it is a **prototype-based** language. This means that JavaScript objects do not inherit from classes. Instead, JavaScript objects can inherit from other objects. When one object inherits properties from another object, that object that is being inherited from is called the prototype object.

This prototype object serves as a blueprint for all other objects that inherit from that object.

```javascript
  function Fruit() {}

  Fruit.prototype.color = "Yellow";
  Fruit.prototype.sweetness = 7;
  Fruit.prototype.fruitName = "Generic Fruit";
  Fruit.prototype.nativeToLand = "USA";
  Fruit.prototype.showName = function() {
    console.log("This is a " + this.fruitName);
  }
```

# Accessing Properties on an Object

There are 2 ways of accessing a property from an object:

  1. Dot Notation
  2. Bracket Notation

## Dot Notation

You will probably use dot notation to access most properties on objects. It follows the simple pattern of `object.propertyName`:

```javascript
  var book = { name: "random book name", genre: "suspense" };
  console.log(book.name); // "random book name"
  console.log(book.genre); // "suspense"
```

## Bracket Notation

Bracket notation can be a useful way of accessing a property name. When accesssing a property, the `key` of every property ultimately converts to a `String` data type. With bracket notation, you can pass in any _expression_ that returns a string.

```javascript
  var banana = { color: "Yellow" };
  var colorProp = "color";

  console.log(banana[colorProp]); // "Yellow"
  console.log(banana["col" + "or"]); // "Yellow"
```

# Own vs Inherited Properties

Let's say you have an object `kiwi` that was created from a `Fruit()` constructor function. That `kiwi` object has already inherited properties and methods from the `Fruit` prototype that were already defined. But you can also define methods on the specific object of `kiwi` that would not be available on the `Fruit` prototype.

```javascript
  var kiwi = new Fruit("green", 6, "Kiwi", ["Africa", "Australia"]);
  kiwi.seedCount = 22;
  kiwi.howManySeeds = function() {
    console.log("This " + this.name + "has approximately " + this.seedCount + " seeds!");
  };

  kiwi.howManySeeds(); // "This Kiwi has approximately 22 seeds!"
```

In this kiwi's case, `howManySeeds` would be the `kiwi`'s **own property** as opposed to `showName` which is a property that `kiwi` inherited from the `Fruit` prototype.

This was a brief overview of the key parts of working with objects in JavaScript. Hopefully this provided a decent explanation of what an object is, and how you can start working with them and harnessing their powerful features.

<p align="center">
  <img height="300" src="https://media.giphy.com/media/13ExF2qUe1ZUbe/giphy.gif" alt="powerful">
</p>