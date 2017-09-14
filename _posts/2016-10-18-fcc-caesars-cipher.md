---
published: true
layout: post
title: "More Projects, More Podcasts..."
author: "Sergio"
permalink: /2016/10/18/fcc-caesars-cipher
date: '2016-10-18 16:16:01 -0600'
---

For my next challenge on FCC I will be working with Caesar’s Cipher. I have been pulling my hair out on how close I feel I am to solving this problem but for some reason cannot get it.

I figured this would be a good time to break this problem down in writing to easily understand while simultaneously working on my blogging skills. 😉

This challenge is about decoding a particular cipher, the ROT13 cipher which simply takes a letter, and shifts it 13 places, so A -> N, B -> O, etc.

In order to shift these capital letters we would take their ASCII code character by converting the character using String.prototype.charCodeAt() (i.e. A -> 65, B ->  66, C -> 67, etc.) and add 13, and then translate that charCode back into a letter using String.fromCharCode()

Seems simple enough right? Just take each letter and jump 13 spots in the alphabet? Well I encountered a couple problems along the way…

When you get to N and beyond, jumping 13 spots forward would result in a character that is not a letter
For example, the ASCII code for Q is 81, adding 13 would result in 94, translating into the ^ character
So, each capital letter is already “assigned” another, meaning that the second half of the alphabet would have to jump back 13 spaces instead of forward to avoid any error or confusion
The order of if statements is important, when I first wrote this code it would not run properly, and only the first number of the codeArray was being manipulated and the following numbers were not firing
So I have created two variables right off the bat, codeArray and finalOutcome. codeArray is an empty array that will store the code characters and take care of the addition/subtraction of the code, while finalOutcome is an empty string that will push the decoded characters into.

```javascript
function rot13(str) {
 var codeArray = [];
 var finalOutcome = "";
}
 ```

Next, I’ll take the ASCII charCode of each letter in the passed string, and push it to an array.

```javascript
for (var i=0; i < str.length; i++) {
  var x = str.charCodeAt(i);
  codeArray.push(x);
}
```

Now I’ll use a series of else if statements to determine whether to add or subtract 13 from the charCode, or leave as is otherwise.

```javascript
for (var j=0; j < codeArray.length; j++) {
  if (codeArray[j] < 65 || codeArray[j] > 90) {
    codeArray[j] += 0;
  } else if (codeArray[j] < 78) {
    codeArray[j] += 13;
  } else if (codeArray[j] < 91) {
    codeArray[j] -= 13;
  }
}
```

Now I’ll use fromCharCode to translate the code into readable characters, and push them into the finalOutcome string.

```javascript
for (var k=0; k < codeArray.length; k++) {
  var z = String.fromCharCode(codeArray[k]);
  finalOutcome += z;
}
```
Finally when we return finalOutcome we get the decoded cipher!

A couple of things I still need work on:

1. Unnecessary storage of variables. I store everything in variables because it seems easier to me to work with, but whenever I check other code I see others going straight to the return function to bypass storage of variables. In due time I will get used to omitting unnecessary variables but it is currently helping my learning process to do otherwise

2. I definitely need to write out these problems more. I was really struggling trying to do everything in my head before, but breaking it down and writing everything out in english before I actually wrote some code was crucial to my success. I think I can get used to this.
