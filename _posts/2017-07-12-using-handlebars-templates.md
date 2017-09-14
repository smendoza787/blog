---
published: true
layout: post
date: '2017-07-12 16:16:01 -0600'
permalink: /2017/07/12/using-handlebars-templates
title: "Using Handlebars Templates"
author: "Sergio"
---

As I've been going through the Javascript section of learn, I've been learning how to use a really neat JavaScript library called [Handlebars](http://handlebarsjs.com/).

Handlebars is awesome because it allows you to create templates out of plain HTML and interpolate the variables you pass into the handlebars template using a JavaScript Object.

Handlebars was a little tough to grasp at first because there was so much going on. I was able to understand it by breaking it down and realizing that everything should be extracted out into its own function to handle all of the handlebars magic.

The first thing you need to worry about is creating the actual Handlebars template. Typically inserted as a script tag with an ID to grab the element later, and a type of "text/x-handlebars-template

```
  <script id="template" type="text/x-handlebars-template">
    <div class="entry">
      <h1>{{title}}</h1>
      <div class="body">
        {{body}}
      </div>
    </div>
  </script>
```

Now we've made a template but we're not done yet. In order for the handlebars magic to take place, we need to grab the HTML string inside out script tag and compile it to a handlebars template by using Handlebars.compile.

```javascript
  var source   = $("#entry-template").html();
  var template = Handlebars.compile(source);
```

Handlebars.compile() will take all of the HTML text in our template and compile it into a usable template and return it as a function.

So we're left with a `template()` function that we can pass an object to, and have those object properties passed into our template.

```javascript
  var context = {title: "My New Post", body: "This is my first post!"};
  var html    = template(context);
```

Our `html` variable will return:

```html
  <div class="entry">
    <h1>My New Post</h1>
    <div class="body">
      This is my first post!
    </div>
  </div>
```

Now then we can do whatever we please with our "blog post" template without having to manually concatenate with JavaScript!
