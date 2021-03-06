---
published: true
layout: post
title: "Tracking My Hikes With A Rails App"
author: "Sergio"
date: '2017-06-23 16:16:01 -0600'
permalink: /2017/06/23/hyyker-rails-app
---

Hiking is one of my favorite hobbies, along with just being outside in general. For my first big Rails project, I wanted to take an activity I really enjoy and quantify the data for other users to see.

This application is built completely with Ruby on Rails, using Devise for user authentication, and bootstrap for the front-end.

The application consists of 3 main models, Users, Trails, and Hikes. The Hike model serves as a join table for a User and a Trail. The application keeps track of a Users Hikes along with their total distance hiked, and total elevation climbed.

To make use of nested resources, I nested all hike creation aspects inside of the trail resource, because why would you created a hike without having a trail to hike anyway? This way, during the creation of the hike, I can keep tabs on the trail_id throughout the process.

To make use of class-level methods we have categories for the most active hiker (most hikes under their belt), the rock climber (highest elevation), and the trail-runner (longest distance hiked) that dynamically change according to who holds those current records.

Thanks to Devise, user authentication was very simple and straight forward. Devise also comes wth everything needed to provide OmniAuth login via Facebook and any other social login.

I had a lot of fun on this project because hiking/exercising is something I really enjoy doing, so I wanted to make sure this app had all the features I would want when using an outdoor hike tracker.

In the near future I would like to update the app to have the ability to upload photos for each trail, have a comment/discussion section for each trail, and maybe even a dynamic leaderboard (possible addition for the jQuery front end section?).
