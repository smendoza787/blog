---
published: true
layout: post
title: "Building A Sinatra App For Music Festivals"
author: "Sergio"
date: '2017-05-13 16:16:01 -0600'
permalink: /2017/05/13/festipal-sinatra-project
---

# [](#header-2) A web app for keeping track of your favorite music festivals!
For my Sinatra project I decided to build something that I was interested in and passionate about.
I built an app for keeping track of your favorite music festivals and their lineups.

# [](#header-2) Demo
<iframe width="560" height="315" src="https://www.youtube.com/embed/-9bKugZrx5k" frameborder="0" allowfullscreen></iframe>

A quick rundown of the technologies used in this project:
- Ruby
- Sinatra
- Rack
- ActiveRecord
- SQL
- Bootstrap
- HTML
- CSS
- Heroku / PostgreSQL (for deployment)

Learn did a great job of preparing me for a bigger Sinatra project by challenging me to build multiple Sinatra apps before appraoching this project (Playlister, "Fwitter"), So I had a pretty good general sense of what I was doing.

# [](#header-2) Database Planning

The first thing I wanted to do was plan out my database structure, and to do that, I knew I was going to have to figure out the associations between the objects I was creating.

I wanted 3 main objects:
- Festivals
  - has_many users
  - has_many artists
- Users
  - has_many festivals
- Artists
  - has_many festivals

I wanted everything to be centered around Festivals. You wouldn't really "Create an Artist" unless you knew which festival they were going to be at. This is Festipal, not Artistpal or Spotify. We'll go through the models and their associations more in the models section.

In order to create the database tables I wanted, I ran the following migrations:

```ruby
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :email
      t.string :username
      t.string :password_digest
    end
  end
end

class CreateArtists < ActiveRecord::Migration[5.1]
  def change
    create_table :artists do |t|
      t.string :name
      t.string :genre
    end
  end
end

class CreateFestivals < ActiveRecord::Migration[5.1]
  def change
    create_table :festivals do |t|
      t.string :name
      t.date :date
      t.string :location
      t.integer :created_by_user_id
    end
  end
end
```

# [](#header-2) Models

The next step was creating the models for my project. This is where things got a little slippery because of my has_many, :through object associations.

```ruby
class Festival < ActiveRecord::Base
  validates :name, presence: true, uniqueness: true
  validates :location, presence: true
  validates :date, presence: true

  has_many :festival_artists
  has_many :artists, through: :festival_artists
  has_many :festival_users
  has_many :users, through: :festival_users
end

class Artist < ActiveRecord::Base
  has_many :festival_artists
  has_many :festivals, through: :festival_artists
end

class User < ActiveRecord::Base
  validates :email, presence: true, uniqueness: true
  validates :username, presence: true, uniqueness: true
  validates :password, presence: true

  has_secure_password

  has_many :user_festivals
  has_many :festivals, through: :user_festivals
end
```

All of my associations worked as planned, except for my festival having many users. Users were allowed to have many festivals, but I wanted to implement a feature where you would be able to go to a festival page and see all of the users that were going to that specific festival. Unfortunately I kept running into errors that were requiring me to build multiple database tables, so I decided to forgo that part of the project, FOR NOW. I know I'll be back later to fix it up. As a matter of fact I thought of a ton more features to add once I considered the project "complete", lol.

But here are the models I made for my has_many, :through objects associations:

```ruby
class FestivalArtist < ActiveRecord::Base
  belongs_to :festival
  belongs_to :artist
end

class FestivalUser < ActiveRecord::Base
  belongs_to :user
  belongs_to :festival
end
```

#[](#header-2) Controllers

The controllers were probably the easiest part of the project for me, and also very repetitive. Every model has a new, edit, show and index page. While I did not make an index for users, like I said I wanted everything to be focused on the festivals.

The most difficult part was probably figuring out the `flash` messages (the messages that flash once on a page when an action is completed). It took me awhile to figure out why some of my messages were not appearing, and it had to do with the number of redirects you sent to the client. If you set a flash[:message] and redirected to a certain page, then that page redirected you to ANOTHER page, the flash would "show" and dissappear on that first redirect. So by the time the app got to the next redirect the flash message would be gone.

I was surprised the rack-flash gem had nothing about it in their documentation, but I ended up figuring it out in the end.

In respect to not flooding this blog post with every line of code in my app, here is my `festival_controller` which is the beefiest one:

```ruby
class FestivalsController < ApplicationController

  get '/festivals' do
    if !logged_in?
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    else
      erb :'/festivals/index'
    end
  end

  get '/festivals/new' do
    if !logged_in?
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    else
      session[:artist_num] = 1

      erb :'/festivals/new'
    end
  end

  post '/festivals' do
    fest = Festival.new(params[:festival])

    if fest.save
      fest.created_by_user_id = current_user.id
      current_user.festivals << fest
      fest.save

      params[:new_artist].each do |artist_params|
        if artist_params[:name] != "" && artist_params[:genre] != ""
          new_artist = fest.artists.find_or_create_by(artist_params)

          if !new_artist.save
            flash[:error] = fest.errors.full_messages

            redirect '/festivals/new'
          end
        end
      end

      flash[:message] = "Successfully created festival."

      redirect "/festivals/#{fest.id}"
    else
      flash[:error] = fest.errors.full_messages

      redirect '/festivals/new'
    end
  end

  get '/festivals/:id' do
    if logged_in?
      @festival = Festival.find_by(id: params[:id])

      if @festival
        erb :'/festivals/show'
      else
        flash[:message] = "Festival does not exist."

        redirect '/festivals'
      end

    else
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    end
  end

  get '/festivals/:id/edit' do
    if logged_in?
      @festival = Festival.find_by(id: params[:id])

      if @festival
        erb :'/festivals/edit'
      else
        flash[:message] = "Festival does not exist."

        redirect '/festivals'
      end

    else
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    end
  end

  get '/festivals/:id/add' do
    if logged_in?
      @festival = Festival.find_by(id: params[:id])

      if @festival
        current_user.festivals << @festival

        flash[:message] = "You're going to #{@festival.name}!"

        redirect "/festivals/#{@festival.id}"
      else
        flash[:message] = "Festival does not exist."

        redirect '/festivals'
      end

    else
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    end
  end

  get '/festivals/:id/remove' do
    if logged_in?
      @festival = current_user.festivals.find_by(id: params[:id])

      if @festival
        current_user.festivals -= [@festival]

        flash[:message] = "You're not going to #{@festival.name} anymore."

        redirect "/festivals/#{@festival.id}"
      else
        flash[:message] = "Cannot find festival in your festival list."

        redirect '/festivals'
      end

    else
      flash[:message] = "You must be logged in to do that."

      redirect '/login'
    end
  end

  patch '/festivals/:id' do
    @festival = Festival.find_by(id: params[:id])
    @festival.update(params[:festival])

    flash[:message] = "Successfully updated festival."

    redirect "/festivals/#{@festival.id}"
  end

  delete '/festivals/:id' do
    @festival = Festival.find_by(id: params[:id])
    @festival.delete

    flash[:message] = "Successfully deleted festival."
    redirect '/festivals'
  end

end
```

#[](#header-2) Views

The views were probably my favorite part of the entire project. I've fooled around with HTML and CSS pretty frequently throughout my life and I really enjoy creating visual experiences.

Each model had the corresponding files for the controller routes.

- New
- Edit
- Index
- Show

I won't go into too much detail about the views because it would probably be better for you to just look at the app yourself!

Then came the process of pushing it to Heroku. It broke around 50 times when I first tried to deploy it. Luckily a fellow Learn student wrote a blog about the subject and luckily I was able to figure the rest out on my own!

I had a lot of fun making this project and I can't wait to learn Rails and build something even better.

Thanks for reading!
