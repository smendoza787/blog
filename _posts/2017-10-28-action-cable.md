---
published: true
layout: post
title: "Building a Chat App With Rails 5 and ActionCable"
author: "Sergio"
permalink: /2017/10/28/chat-app-actioncable
date: '2017-10-28 16:16:01 -0600'
---

A few months ago Reddit released a short-lived experiment of sorts called r/place. The idea of the project involved 72 hours where people would be able to place a single pixel on this giant 10000px x 10000px blank canvas once every 10 minutes. 

This was a really interesting experience to live through and be apart of. People had to come together in creative ways in order to achieve an awesome drawing. Some groups teamed up to draw murals of their cities, sports teams, and subreddits while others teamed up to 'destroy' or draw over the beautiful work of others.

You can check out the [time lapse of the project here](https://www.youtube.com/watch?v=XnRCZK3KjUY).

As cool and insightful as that project was, I think the underlying technology behind it is even cooler.

# WebSockets

r/place used a technology called WebSockets, which is defined from Wikipedia as follows:

> The WebSocket protocol enables interaction between a browser and a web
> server with lower overheads, facilitating real-time data transfer from
> and to the server. This is made possible by providing a standardized way
> for the server to send content to the browser without being solicited by
> the client, and allowing for messages to be passed back and forth while
> keeping the connection open. In this way, a two-way (bi-directional)
> ongoing conversation can take place between a browser and the server.

As a result of WebSockets, gone are the days of waiting for a request from the client to send a response to the server. As soon as your browser 'upgrades' to the websocket connection, you are in a constant "handshake" with that server, and will receive unsolicited responses pushed to your browser in real-time.

In practice, Reddit was able to develop a web application that established websocket connections with hundreds of thousands of people across the globe. Whenever someone sent a POST request with their color and coordinates, the server sent out responses to browsers that updated the canvas with JavaScript.

# ActionCable Intro

Rails 5 introduced a new feature called ActionCable, designed to integrate WebSockets easily into any Rails application. We can now integrate real-time features and write in the same style that we write the rest of our Rails projects!

ActionCable provides a client-side JavaScript framework, along with a server-side Ruby framework. This gives us access to our Ruby models and ActiveRecord or another ORM of our choice.

## Pub/Sub

The first concept to understand is the [Publish/Subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) pattern.

The Pub/Sub system involves the sender of data, aka the Publisher, categorizing published messages into classes without knowing which Subscribers are subscribed to it.

Similarly, subscribers can subscribe to as many classes as they wish and only receive messages that they choose to receive messages from.

# Server-Side Components

## Connections

Connections form the foundation of the client-server relationship. Every time a Websocket is accepted by the server, a connection object is instantiated.

The client of a WebSocket connection is called a consumer. Anyone can create a consumer-connection pair per browser tab/window/device they have open.

## Connection Setup

I originally planned on using Devise for my authentication until I ran into some issues getting the `session[:user_id]` because of the way Devise handled things, so I found the [Clearance gem](https://github.com/thoughtbot/clearance) and ended up loving how simple it was to setup and how well Thoughtbot documented it all.

The code in my `connection.rb` is in charge of finding the right room to connect in my `ChatChannel`, along with some other code that involved using Clearance models to find the `current_user`:

```ruby
# app/channels/application_cable/connection.rb

module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_current_user
      reject_unauthorized_connection unless self.current_user
    end

    private
      def find_current_user
        if remember_token.present?
          @current_user ||= user_from_remember_token(remember_token)
        end

        @current_user
      end

      def cookies
        @cookies ||= ActionDispatch::Request.new(@env).cookie_jar
      end

      def remember_token
        cookies[Clearance.configuration.cookie_name]
      end

      def user_from_remember_token(token)
        Clearance.configuration.user_model.find_by(remember_token: token)
      end
  end
end
```

## Channels

Channels behave similarly to controllers in MVC environments. They encapsulate a logical unit of work, and contain shared logic between your channels.

For example, you could have a ChatChannel for chat functionality, and an Appearance Channel for the ability to see when people log in/log out. A consumer can be subscribed to either or both of these channels at once.

## Streams

Streams provide the avenue by which channels route published content to their subscribers. All messages that are broadcast to a stream will be visible to all users subscribed to that stream. 

In our case, a chat should be private to the 2 users involved in that chat. However I would like to later implement a chat room where multiple users can subscribe to a chat room stream.

## Subscriptions

When a consumer subscribes to a channel, they are now considered a subscriber of that channel. The connection between a channel and subscriber is known as a subscription.

When a message is broadcast from the channel, it delivers that message to all subscribers of that channel based on an identifier sent by the consumer.

## Code

Below is the code for my `ChatChannel` involving channels, streams and subscriptions:

```ruby
# app/channels/chat_channel.rb

class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_#{params['chat_id']}_channel"
  end

  def unsubscribed
  end

  def send_message(data)
    current_user.messages.create!(body: data['message'], chat_id: data['chat_id'])
  end
end
```

The `subscribe` and `unsubscribe` methods get called when a client subscribe and unsubscribes to the chat channel respectively.

When a new subscription is created, the server calls ActionCable's `stream_from` method in order to stream all messages from the `"chat_#{params['chat_id']}_channel"` which means each stream is private to each user involved.

# Client-Side Components

## Connections

A Consumer requires an instance of the connection on their side, this is established using some client-side code that Rails already generates for you for every new project:

```ruby
# app/assets/javascripts/cable.js

//= require action_cable
//= require_self
//= require_tree ./channels


(function() {
  this.App || (this.App = {});


  App.cable = ActionCable.createConsumer();
}).call(this);
```

This code connects against `/cable` on your server by default. However, the connection won't be established until you specify at least one subscription you'd like to have.

## Subscriber

A consumer becomes a subscriber by creating a subscription to a given channel.

```ruby
  App.cable.subscriptions.create { channel: "ChatChannel", room: "Best Room" }
```

A consumer can act as a subscriber to a given channel any number of times.

## Code 

Below I've thrown in some of the code I used for the client side part of my application. When the page is first loaded, a subscription is made to the `ChatChannel`, and jQuery takes care of dynamically rendering the received content onto the page.

```javascript
// app/assets/javascripts/channels/chat.coffee

jQuery(document).on 'turbolinks:load', ->
  $messages = $('#messages')
  $message_body = $('#message_body')
  $message_submit = $('#message_submit')
  if $messages.length > 0
    messages_to_bottom = ->
      $('#chat-messages').scrollTop($('#chat-messages').prop("scrollHeight"))
    messages_to_bottom()
    App.chat = App.cable.subscriptions.create {
      channel: "ChatChannel"
      chat_id: $messages.data('chat-id')
    },
    connected: ->
      # Called when the subscription is ready for use on the server

    disconnected: ->
      # Called when the subscription has been terminated by the server

    received: (data) ->
      if data['message']
        $messages.append data['message']
        messages_to_bottom()

    send_message: (message, chat_id) ->
      @perform 'send_message', message: message, chat_id: chat_id

    $(document).keypress (e) ->
      if e.keyCode is 13 && $message_body.is(":focus")
        App.chat.send_message $message_body.val(), $messages.data('chat-id')
        e.preventDefault()
        $message_body.val('')

    $message_submit.click ->
      App.chat.send_message $message_body.val(), $messages.data('chat-id')
      $message_body.val('')
      e.preventDefault()
```

# View Templates

I don't want to get into too much detail about the views and templates. Aside from the WebSocket functionality, it's essentially a messaging CRUD app with an index and show page for chats, and some extra views that overwrite the default Clearance signin/signup forms:

![Imgur](https://i.imgur.com/JUkukJv.png)

That, and a whole lot of SCSS which was my first real go at using it in a Rails environment (_side-note: SO CONVENIENT_):

![Imgur](https://i.imgur.com/U5mhdNv.png)

## Issues With Rendering Messages

I do want to focus on a small issue I was having with the `_message` partials rendering the way I wanted them to.

I wanted the current_user's messages to render with a different color in order to differentiate the message from the other user's message, kind of like a normal chat app.

```ruby
  # app/views/messages/_message.html.erb

  <% if current_user.id == message.user.id %>
    <div class="message">
      <div class="message-user">
        <i class="fa fa-user-circle fa-2x" aria-hidden="true"></i>
        <p><%= message.user.name %></p>
      </div>
      <div class="message-content">
        <p><%= message.body %></p>
      </div>
    </div>
  <% else %>
    <div class="message">
      <div class="message-user other-user">
        <i class="fa fa-user-circle fa-2x" aria-hidden="true"></i>
        <p><%= message.user.name %></p>
      </div>
      <div class="message-content">
        <p><%= message.body %></p>
      </div>
    </div>
  <% end %>
```

Initially, the `chat#show` page was rendering each chat message with:

```ruby
  # app/views/chats/show.html.erb

  <div id="chat-messages">
    <div id="messages" data-chat-id="<%= @chat.id %>">
      <%= render @chat.messages %>
    </div>
  </div>
```

The problem with this, is that the message partial does not have access to the `current_user`, so I kept running into errors.

## Rendering Messages Solution

My solution involved creating a separate message partial for the "other user", and iterating over all of the chat messages while conditionally rendering each message based on whether the message user was the current user or not. I also had to explicitly pass in the message local variable.

```ruby
  <div id="chat-messages">
    <div id="messages" data-chat-id="<%= @chat.id %>">
      <% @chat.messages.each do |msg| %>
        <% if msg.user == current_user %>
          <%= render partial: 'messages/message', locals: { message: msg } %>
        <% else %>
          <%= render partial: 'messages/othermessage', locals: { message: msg } %>
        <% end %>
      <% end %>
    </div>
  </div>
```

This allowed me to access the `current_user` variable, and compare `current_user.id` and `message.user.id` in order to apply different classes to the message div.

# Live Demo

You can see a [live demo of the app here](https://bp-messenger.herokuapp.com).

I hope this intrigued you enough to take a look at ActionCable and try to figure out how to create your own real time apps. A chat feature is one of a ton of possibilities. Next, I hope to learn enough Docker to be able to deploy my own container environents and write up a post on why the container movement is changing the way we develop applications.