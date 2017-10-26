---
published: true
layout: post
title: "I'm Learning Sass and I Don't Think I'll Ever Go Back"
author: "Sergio"
permalink: /2017/10/23/learning-sass
date: '2017-10-23 16:16:01 -0600'
---

After getting more comfortable with moving around in a codebase, I started noticing a lot of little annoyances that slowed me down when I was getting into a really good flow. Things like having to change multiple values, not being able to find certain CSS selectors, or not being able to select certain elements so easily. These are some of the problems I have with CSS:

# CSS Doesn't Use Variables

If you assign a color to a few elements, say a navigation and a footer, every time you change that color in one place you now need to change that color in every other element to keep things consistent.

```css
.navigation {
	background-color: #ff0000;
	display: flex;
	justify-content: space-between;
}

.some-div {
	background-color: #ff0000;
	font-size: 2.2em;
	color: #cccccc;
}

.other-similarly-colored-div {
	background-color: #ff0000;
	border-radius: 5px;
}
```

If these 3 divs were on completely different stylesheets, it would be tiring to change each value everytime you wanted to change that color.

# CSS Doesn't Let You Nest Selectors

Another tedious task in CSS involves working with specificity. Given the HTML:

```
<div class="contact-info">
	<h2>Mo Schmo</h2>
	<p>This is a contact card for Mo Schmo</p>
</div>
```

If you wanted to style the `h2` and `p` tags differently, the styling for the `.contact-info` might look like this:

```css
.contact-info {
	background-color: #579f678;
}

.contact-info h2 {
	font-size: 2em;
	color: pink;
}

.contact-info p {
	font-weight: 300;
	color: yellow;
}
```

We just had to write out 3 separate selectors for content contained in a single `div`. It would be alot easier to select the contact-info and all the elements inside of it by using some sort of nesting technique.

# CSS Can Get Messy

The bigger your codebase, the more you need to keep it organized. Up until recently my stylesheets were pretty small, never surpassing more than 30 or 40 selectors. Once I started working on projects that had a lot more components involved, I found myself scrolling up and down my single stylesheet looking for the right selector. This made me wish I had a better system of organization.

It would be neat if CSS had a way to organize your stylesheets in a tidy file structure.

# Let's Get SASSy

For this post we'll be addressing SCSS (Sassy CSS) which is the newer version of SASS, the main difference being SASS uses indentation for a clean look, and SCSS uses semi-colons and curly brackets for a more traditional feel.

SASS is a precompiler for CSS. This means that everything written in SASS gets compiled into regular CSS when ran through the Sass compiler.

This post won't go over the compiling stage too much, the documentation goes over that pretty thoroughly. Just know that most frameworks and build tools like Rails and Gulp have support for Sass built-in. Even for the sake of this post, SASS itself will compile your `.scss` files for you if you install and run something like `sass main.css output.css`.

Sass addresses the common pitfalls of writing regular CSS, and lets you write cool-looking CSS in their special syntax that compiles into regular CSS.

Lets go over their solutions for the problems we described earlier:

# SCSS Uses Variables

Sass uses the concept of variables. For any value that we plan on using a lot throughout our code (fonts, colors, whatever), we can simply store them in a variable for later use:

```css
$header-font: 'Open Sans', sans-serif;
$favorite-color: #ff00ff;

.header {
	font-family: $header-font;
	color: $favorite-color;
}

h4 {
	color: $favorite-color;
}
```

Now if we need to change our favorite color, we can change the value in one place, super neat!

# SCSS Also Uses Nesting

Nesting is a concept that we should already be used to seeing in HTML. Seeing elements nested inside of each other is a great way to visualize your elements.

Sass allows you to nest your selectors inside of each other as a way of organizing your CSS and making it more readable. 

```css
	.parent-div {
		background-color: green;

		h1 {
			color: blue;
		}

		h2 {
			color: orange;
		}
	}
```

The above gets processed into:

```css
.parent-div {
	background-color: green;
}

.parent-div h1 {
	color: blue;
}

.parent-div h2 {
	color: orange;
}
```

This is a mundane example, but using nested selectors has definitely increased my productivity.

# SCSS `@import`s Stuff For You

CSS currently has an `@import` option that allows you to separate your files into manageable chunks, but the only drawback is that it creates an HTTP request every time it is called. Sass builds on top of CSS `@import` but instead of requiring an HTTP request, Sass will take the file you want to import and combine it with the file you're importing into so you can serve a single CSS file to the web browser.

Let's say you had a couple of files that you wanted to import, you could write:

```scss
// _reset.scss

html,
body,
ul,
ol {
  margin:  0;
  padding: 0;
}

----------------------------------------

// base.scss

@import 'reset';

body {
  font: 100% Helvetica, sans-serif;
  background-color: #efefef;
}
```

This is just the tip of the iceberg when talking about the abundance of CSS preprocessors out there ( [less](http://lesscss.org/), [stylus](http://stylus-lang.com/), [postcss](http://postcss.org/) ).

I highly recommend taking a look at all of them and finding one that resonates with you. Most of them should have an area for you to try out the language to see if its something you enjoy working with.