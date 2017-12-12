---
published: true
layout: post
title: "Let's Talk About The Spread Operator"
author: "Sergio"
permalink: /2017/12/11/spread-operator
date: '2017-12-11 16:16:01 -0600'
---

We're now at the bitter-sweet end of 2017 and it's been a wild year for JavaScript. A lot of new frameworks came out, a lot of new courses came out, and a lot more people have decided that they want to pursue the amazing career of web development. That means that more people are starting to learn JavaScript!

I think by now, it's safe to say that most people have migrated toward the newer [EcmaScript6](https://www.ecma-international.org/ecma-262/6.0/) Syntax when writing JavaScript, and I wanted to highlight one of the new features it includes called the *spread operator*.

## Definition

*Spread syntax* (`...`) allows an iterable, such as an array or string, to be expanded in places where zero or more arguments (for function calls) or elements (for array literals) are expected, or an object expression to be expanded in places where zero or more key-value pairs (for object literals) are expected.

## Use Cases

The spread operator comes in handy when you want to "expand" an array or object and its properties without having to replicate the entire object by hand.

A few examples include:

### Function Calls

```javascript
  function someFunc(a, b, c) {
    return a + b + c;
  }

  let someObject = [2, 4, 6]

  someFunc(...someObject); // someFunc(2, 4, 6)
```

This certain case destructures the object inside of the function call, and allows the object properties to be used as arguments without having to type it out by hand.

### Array Literals

```javascript
  const someArr = [3, 5, 1];
  const spreadArr = [...someArr, '4', 'six', 8];

  spreadArr; // [3, 5, 1, '4', 'six', 8]
```

Using the spread operator within an array literal, allows you to spread the contents of `someArray` inside of the new array object, resulting in a flattened array containing the "spread" contents of the original array.

### Object Literals

```javascript
  let iceCreamCone = {
    price: 1.5,
    chocoDipped: false
  }

  let myCone = { ...iceCreamCone, sprinkles: true }

  myCone; // { price: 1.5, chocoDipped: false, sprinkles: true }
```

When using the spread operator with object literals, it behaves like an array where the inner contents of the "spread" object get dispersed on a flattened level within the new object literal.

### Copying an Array

```javascript
  let arr1 = [3, 7, 6];
  let arr2 [...arr1];

  arr2.push(2); // [3, 7, 6, 2]
  arr1; // [3, 7, 6] arr1 is unaffected
```

### Concatenating Arrays

```javascript
  let arr1 = [0, 1, 2];
  let arr2 = [3, 4, 5];

  let arr3 = [...arr1, ...arr2]; // [0, 1, 2, 3, 4, 5]
```

### Merging Objects

```javascript
  let obj1 = { foo: 'bar', x: 34 };
  let obj2 = { foo: 'baz', y: 67 };

  let clonedObj = { ...obj1, ...obj2 }; // { foo: 'baz', x: 34, y: 67 }
```

Notice how the original `foo: 'bar'` is overwritten by the second object which has its `foo` property set to `'baz'`. The final property takes on the value of whatever it was set to last.

## Conclusion

These are just some of the convenient ways you can use the new ES6 Spread operator. I've found great use of it when writing components in React, and creating different reducers in Redux. I hope on diving deep into additional ES6 and potentially ES7 features since we are closing in on the release of ES8.