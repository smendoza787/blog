---
published: true
layout: post
title: "Getting Modular With CSS in React"
author: "Sergio"
permalink: /2017/12/04/getting-modular
date: '2017-12-04 16:16:01 -0600'
---

I've been working with React for a while now, and one thing that always perplexed me was file structure organization. As the sole developer of my side projects, I never had a style-guide to follow when structuring my code.

As a result, my project's file structures have always been a bit of a mess. I was so hyper-focused on my React components that I paid little attention to where my CSS files would end up. This lack of TLC to my CSS files ended up in a hard-to-read mess of CSS.

I felt that there was a solution somewhere, so I decided to hit GitHub hard and peruse through the source code of random React projects.

## Solution

I ended up finding a solution that I really liked, and followed the pattern of modularity that React is most-loved for.

A well-done component should be a piece of code that contains all of it's dependencies, and has the ability to be inserted into (or removed from) any project without breaking the rest of your code. This includes your CSS!

My solution was to start treating components as all-inclusive folders. These folders would include the component itself, the CSS for that component, and any exclusive dependencies it might need.

## Example

For example, let's take a simple Header component. It'll render a simple H1 header with text and styling.

I've created a folder with the name of the component (`Header`) inside of my components folder. Within this folder, I'll be putting the actual component, the CSS file it depends on, and any other dependencies it might need.

```javascript
  // /src/components/Header/index.js

  import React from 'react';
  import './index.css';

  class Header extends Component {
    render() {
      return (
        <h1 className="main-header">Hello World!<h1>
      )
    }
  }
```

For the CSS file, we insert it into our `Header` directory:

```css
  /* /src/components/Header/index.css */

  .main-header {
    background-color: red;
    font-size: 2em;
  }
```

This works perfectly, because we can simply import the component folder itself, and it will automatically import the `index.js` file inside of our component's folder by default.

If we wanted to render our `Header` component inside of our `App` component, we could simply insert it like so:

```javascript
  // /src/App.js

  import React, { Component } from 'react';
  import Header from './components/Header';

  class App extends Component {
    render() {
      return (
        <Header />
      )
    }
  }
```

## Conclusion

As a result of this, we can use this component folder structure to drop folders in whatever project we want, and that folder will contain the component, it's own CSS and any dependencies. Amazing!