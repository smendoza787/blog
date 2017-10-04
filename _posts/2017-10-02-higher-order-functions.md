---
published: true
layout: post
title: "Going Up With Higher Order Functions"
author: "Sergio"
permalink: /2017/10/02/higher-order-functions
date: '2017-10-02 16:16:01 -0600'
---

Ever since I've been getting a firm grip on JavaScript as a language, I've made it a point to read up more on the intricacies of the languages. Everyone knows about functions in JavaScript, and I always assumed the only 'function' they had was encapsulating blocks of code so they are easily called somewhere else. I guess that's a sky-high overview of what they are, but then I learned that functions are treated as _First Class Objects_.

What are First Class Objects and how are they treated in JavaScript, you ask? *First Class Objects* in JavaScript support all the operational properties inherent to other entities; properties such as being able to be assigned to a variable, passed around as a function argument, returned from a function, etc. Essentially, being a First Class Object in JavaScript means you are able to do what _any object_ in JavaScript can do. Objects like Strings, Integers, Arrays, etc.

With that in mind, we can take a deeper look at *Higher Order Functions* in JavaScript. HOF allow us to get rid of all the verbose for/while loops we see all over the place, and replace it with terser, more succinct code.

We'll look at 3 Higher Order Functions in particular, provided by the JS language:

- Map
- Filter
- Reduce

We'll be working with an array of `pets` to demonstrate each of these functions:

```javascript
  const pets = [
    { name: 'Cosmo', type: 'dog', age: 10 },
    { name: 'Olive', type: 'dog', age: 3 },
    { name: 'Rocket', type: 'cat', age: 1 },
    { name: 'Scaredy', type: 'cat', age: 12 }
  ];
```

## Array.prototype.map()

JavaScript provides us with a function `map` that comes standard on the Array.prototype. Map will take in a callback function as a parameter, and call that function on each element of the array that `map` is initially called on.

What does this callback function look like? Well it takes in a `element` parameter and 2 optional parameters, `index, array`. `element` being the 'current element' that is being iterated over, `index` representing the current index of the `element` being looped over, and `array` representing the entire array that `map` was originally called on.

Let's say that we want to use our `pets` array and create a new array that consisted of the names of our `pets`.

```javascript
  var namesOfPets = pets.map(pet => pet.name);

  console.log(namesOfPets);
  // ["Cosmo", "Olive", "Rocket", "Scaredy"]
```

Map is taking in our callback (`pet => pet.name`) and calling that function on each object in our `pets` array, and telling it to only return the `name` of the pet, instead of the entire pet object.

We could have also asked for a different attribute of each pet to create a different array:

```javascript
  var agesOfPets = pets.map(pet => pet.age);

  console.log(agesOfPets);
  // [10, 3, 1, 12]
```

Conversely, if we had just used the callback (`pet => pet`), it would have returned the entire pet object, resulting in the exact same array.

## Array.prototype.filter()

Another function provided to us by the `Array.prototype` is `filter`. Filter is used when we have an array of items that we want to 'filter' into a new array of items that each pass the test of the callback we provide it. Filter is applied to an array, and uses a callback function that returns true or false to determine which items from the original array will make it into the newly created array.

Lets say we wanted a new list of pets that filters out the older, more senior pets from the younger ones. And just to use a new function we just learned about, lets map the resulting array of older pets to just include the `name` of each pet:

```javascript
  var seniorPetNames = pets
    .filter(pet => pet.age > 8)
    .map(pet => pet.name);

  var youngerPetNames = pets
    .filter(pet => pet.age < 8)
    .map(pet => pet.name);

  console.log(seniorPetNames);
  // ["Cosmo", "Scaredy"]

  console.log(youngerPetNames);
  // ["Olive", "Rocket"]
```

Our `seniorPetNames` variable initially calls `pets.filter` and returns a list of `pet` objects that pass the test of `pet.age > 8`, but that returns a weird looking list of objects (which might be what you want, in some cases), but for readability we chain a `map` function onto that list of pets and specify that we only want the `name`s of each of the pets that passed the initial test in the new array.

`filter` gives us a simple and readable way of transforming an array of items, into an array of items whose properties pass the test we give it via a callback function.

## Array.prototype.reduce()

The last and slightly more involved HOF we'll talk about is `reduce`. Reduce takes the callback function to apply to each item in the array, and an optional initial value to `reduce` each value into. If an initial value is not provided, the first item in the array is used.

The callback function provided takes in 2 arguments, the `prev` value which was returned from the previous item in the list, and the `curr` current item being iterated over.

The most classic use case would be using a Math function to add/subtract/etc. each of the values down to a single numeric value.

Let's try adding up the ages of our pets:

```javascript
  var combinedPetAges = pets
    // first convert array of pet objects into an array of pet ages
    .map(pet => pet.age)
    // reduce pet ages into an initial value, since we didn't provide one reduce will use the first item in the array
    .reduce((prev, curr) => prev + curr)

  console.log(combinedPetAges);
  // 26
```
For out `combinedPetAges` value, we first `map` the `pets` array into an array of pet ages, then we chain on a reduce to work on that return value, which takes the first age in the array (since we did not provide an initial value), and combines it with the next age. The value returned from that function shows up in the next function that is called, as the `prev` argument. This process continues until all values in the list have received that callback function, which results in all of our pet ages added up into the final returned value of `26`.

*Higher Order Functions* and the concept of *First Class Objects* allow us to include functions along with primitive values into our programs, creating more complex and elegant-looking code. We have the ability to pass around, manipulate, and return functions as values to be used by _other_ functions in our programs. These tools are extremely powerful, and hopefully this little post taught you something and will enable you to create more sophisticated programs. :)