---
published: true
layout: post
title: "Building a URL Shortener with React and Rails!"
author: "Sergio"
permalink: /2017/12/24/url-shortener
date: '2017-12-24 16:16:01 -0600'
---

# Throwing Down The Development Gauntlet

Recently I was tasked with creating a URL shortener application in whatever language or framework that I wanted. I was given a mock-up PDF of what the final product was going to look like, along with a folder of jpg resources. I was loving this because I always get hung up on the design of my projects before I even scratch the surface of the logic component of the project. This way, I was given all my tools and a specific end goal and just had to deliver results: an app that would shorten a URL, and would actually redirect to the original URL when pasted into a browser.

# Whats The Plan?

The stack I had in mind was an absolute no-brainer. React and Rails are my bread and butter and this project was no exception. I knew Rails would be perfect to generate the `Link` model I needed that would store the original URL along with it's corresponding "short code". 

React was the perfect choice on the front end because there were a few dynamic elements required in the project (When the short URL is copied, 'Copied!' appears under the input field). Since I have been pumping out React components for the past year I figured I could get this project up and running in no time. :)

# Party In The Backend

The backend of this project is fairly straightforward. In a URL-shortening app, the only model we need is a `Link`. A `Link` has a randomly-generated short `code` that will be used to find the entry in the database, and grab its corresponding `code`.

## Link Object

![link_object](https://i.imgur.com/6pQm05G.png)

```ruby
  class Link < ApplicationRecord
    validates :original_url, presence: true, on: :create
    validates_format_of :original_url,
      with: /\A(?:(?:http|https):\/\/)?([-a-zA-Z0-9.]{2,256}\.[a-z]{2,4})\b(?:\/[-a-zA-Z0-9@,!:%_\+.~#?&\/\/=]*)?\z/
    before_create :generate_code_and_url

    def generate_code
      # create array of all possible letters and numbers
      chars = ['0'..'9', 'A'..'Z', 'a'..'z'].map{|range| range.to_a}.flatten

      # assign value to long_url
      self.code = 6.times.map{chars.sample}.join

      # to check for duplicates we'll generate a new string until it's unique in the DB
      self.code = 6.times.map{chars.sample}.join until Link.find_by(code: self.code).nil?

      # return new short url
      return self.code
    end
  end
```

To create a valid `Link`, all it needs is a valid `original_url` (containing 'http' and '.com'). Upon creation, the `Link.generate_code` method is called, and generates a random string of 6 alphanumeric characters. It will generate a new string until `Link.find_by(code: self.code)` returns `nil`.

I felt the logic for generating the short code was sufficient for this project, as there around 56 billion unique combinations for a code 6 characters long. This wouldn't scale for a project that had millions of users, but for the potential 25+ people that will use this project I thought it would be plenty! Also, adding an extra character to the string would be extremely simple and result in exponentially more combinations.

## API Routes

With our model in place, we just need to make sure that we configure our routes properly and make sure they are RESTful. With this simple project we only have a '/links' endpoint that we'll make POST requests to in order to create our Links.

```ruby
  Rails.application.routes.draw do
    resources :links, only: [:create]
  end
```

I'll admit this is an extremely simple API. I initially thought I could use the Rails API to create a `'/:code` route that would connect to a `Links#Show` controller method that would `redirect_to` the target URL, but I decided to handle the re-routing on the front-end via React Router.

# React Front End

![react](https://i.imgur.com/qJAQyrA.jpg)

Most of the components on the page are presentational and stateless. We have a `background-image` set to `cover` to stretch across the entire page and remain responsive. With the magic of [flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/), I was able to position the presentational components exactly where I wanted them for desktop and mobile.

The two components I want to highlight are the `CodeRedirect` and `UrlShortener` components because that is where 99.99% of the magic happens.

## UrlShortener Component

The _piece de resistance_ of the project is the `UrlShortener` component. It is in charge of keeping track of it's state to determine whether or not it's a good time to make a `POST` request when the 'Shorten URL' button is clicked.

```javascript
  class UrlShortener extends Component {
    constructor() {
      super()

      this.state = {
        input: '',
        output: '',
        isDisabled: true,
        linkSelected: false
      }
    }

    ...
  }
```

We're keeping track of the `input` and `output` fields through state for obvious reasons, and we're keeping track of the `linkSelected` boolean so we can automatically highlight/select the shortlink once a user hits the 'Shorten URL' button.

## _The Shortening_

When our 'Shorten URL' button is clicked, there are a series of things that need to happen.

1. Make sure there is a valid link in the input field
2. Create a `formData` object to send in the body of our `POST` request 
3. Make a `POST` to the `'/links'` endpoint of our API
4. Grab the `code` from the returned created Link object and pipe it into our `output` component state

```javascript
  onShortenClick(event) {
    if (this.state.input.includes('http')) {
      var object = { link: this.state.input }
      var formData = objectToFormData(object)

      fetch('/links', {
        method: 'POST',
        body: formData
      }).then(resp => resp.json())
        .then(linkObj => this.setState({ output: `${window.location.href}${linkObj.code}`, isDisabled: false, linkSelected: true }))
    }
  }
```

Putting our methods into our `UrlShortener` looks like this:

```javascript
  render() {
    return (
      <div className="url-shortener"
          style={{ fontFamily: font }}>
        <h1>Shorten any URL in less than a second.</h1>
        <h2>Make it easier to send and embed links.</h2>
        <UrlInput
          onChange={this.onInputChange.bind(this)}
          longUrl={this.state.input} />
        <UrlOutput
          shortUrl={this.state.output}
          isDisabled={this.state.isDisabled}
          linkSelected={this.state.linkSelected} />
        <MainButton
          linkSelected={this.state.linkSelected}
          onShortenClick={(event) => this.onShortenClick(event)}
          onCopyClick={(event) => this.onCopyClick(event)}
          shortUrl={this.state.output} />
      </div>
    )
  }
```

Now we can generate a short link associated with the URL that we just shortened. Awesome!

## Dynamic Hover Effect Extravaganza

There are still a few dynamic features I needed to implement. One feature was a simple label that reads `Press CMD + C to copy` when you hover over the output field, and then flashes `Copied!` when you actually copy the text.