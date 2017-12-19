---
published: true
layout: post
title: "Making Function Calls Look Pretty With The Pipeline Operator"
author: "Sergio"
permalink: /2017/12/18/pipeline-operator
date: '2017-12-18 16:16:01 -0600'
---

![Imgur](https://i.imgur.com/DSIE5yZ.jpg)

Although the TC39 Comittee only deploys new features to JavaScript every year, it feels like updates to ECMAScript are happening almost every month in the JavaScript world (we're already coming up on ES9!).

I've decided to keep a close eye on the [proposals page](https://github.com/tc39/proposals) on TC39's Github in order to keep track of upcoming features proposed for the next version of JavaScript.

One feature in particular that seems exciting and will surely be more commonly adopted in the coming years is the *pipeline operator*.

_Piping_ is not a new concept by any means. It is very common in UNIX Bash environments as a way to _pipe_ results into the next operation for maximum efficiency.

It has also been a feature in popular functional programming languages such as [F#](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining) and [Elixir](https://elixirschool.com/en/lessons/basics/pipe-operator/) as a way to make functional code look more readable.

# Piping Example

Let's look at an example that will show you what the pipeline operator aims to do for us in JavaScript...

Suppose we have the following functions:

```javascript
  const capitalize = x =>
    x[0].toUppercase() + x.substring(1)

  const doubleSay = x => x + ', ' + x

  const exclaim = x => x + '!'
```

Assuming we will be passing strings into these functions, the function names are pretty self-explanatory as to what they are going to be performing on our strings:

```javascript
  capitalize('wassup') // 'Wassup'

  doubleSay('wassup') // 'wassup wassup'

  exclaim('wassup') // 'wassup!'
```

Now let's say we wanted to apply all 3 functions to our string. It would look a little something like this:

```javascript
  exclaim(capitalize(doubleSay('wassup'))) // 'Wassup, wassup!'
```

If this was the first time we were looking at this code, we may have done a double-take to really understand what was going on. But just to break it down into something we can understand, we're taking the string `'wassup'` and passing it into `doubleSay()`, then passing it into `capitalize()`, and finally passing it into `exclaim()` which leaves us with our final string `'Wassup, wassup!'`.

Due to the 'functional' nature of JavaScript and being able to pass functions into functions, this just works and gives us our desired result at the expense of readability.

# Pipeline Operator To The Rescue

The proposed *[pipeline operator](https://github.com/tc39/proposal-pipeline-operator)* aims to improve readability in situations like ours by borrowing the syntax from common functional programming languages.

```javascript
  let result = exclaim(capitalize(doubleSay("wassup")))
  result // "Wassup, wassup!"

  let result = "wassup"
    |> doubleSay
    |> capitalize
    |> exclaim

  result // "Wassup, wassup!"
```

The pipeline operator (|>) literally "pipes" the result to the left of it, into the function to the right of it. It allows you to follow the chain of functions down to its final result resulting in a readable format!

I think this is great, and allows JavaScript to lend itself nicely to traditional functional programming concepts.

This is all fine and well for functions that only take a single argument, but what happens if we pipe a result into a function that takes multiple arguments?

# Multiple Arguments

```javascript
  const double = x => x + x

  const add = (x, y) => x + y
```

Let's say we first want to `double` a number, and then add the result to another number of our choice. How would the pipeline operator know which argument we want to fill in the next function call? We can use arrow functions to delegate where the result of the pipeline call will be inserted into the next function:

```javascript
  let person = { score: 25 }

  let newScore = person.score
    |> double
    |> (_ => add(20, _))

  newScore // 70
```

We used an underscore (_) here for clarity, but since it is just an arrow function, we could technically use any placeholder we want.

# Conclusion

I think the pipeline operator will be an awesome addition to the next version of JavaScript, and I personally think will allow for an easier learning curve when learning functional programming languages. It also allows the programmer to see how data manipulation is actually handled when dealing with function calls and the results that they return.