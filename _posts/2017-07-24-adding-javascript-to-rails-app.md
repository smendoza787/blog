---
published: true
layout: post
date: '2017-07-24 16:16:01 -0600'
permalink: /2017/07/24/adding-javascript-to-rails-app
title: Adding a JQuery Front End to Hyyker Rails App
author: "Sergio"
---

Here we are again; a month older, a month wiser and another really productive month towards meeting my goals of coming a software developer.

Over the past month I've had the privilege of combining some of my skills and interests into creating an app called Hyyker! I've learned a ton of new stuff about JavaScript, jQuery, and how they fit into the Rails asset pipeline, and with my newfound knowledge I was able to add some interactive features and render content to the page without any page reloads! Lets see how I did it...

With jQuery I've been able to query elements on the page and use AJAX to request and inject the response into elements in the DOM at my whim. I've also learned how to serialize and render JSON with Rails, and consume the data on the front end of the application.

To do this, I had to do a bit of prep in order to convert my Rails app into an API Back-End (with some pages still rendering the `:show` and `:index` views) that served up JSON data upon request to it's endpoints.

# [](#header-2)Serializers

First, I enlisted the help of the `active_models_serializers` gem in order to serialize my responses at the Model level of my application.

```ruby
# /serializers/trail_serializer.rb

class TrailSerializer < ActiveModel::Serializer
  attributes :id, :name, :location, :distance, :elevation, :trail_type
  has_many :users
  has_many :comments
end
```

Now whenever you make a request to the trail route and specify a JSON response, you'll get JSON in return:

![gif](http://i.imgur.com/bB9uM5K.gifv)

# [](#header-2)Controllers

Next, I went through each controller and added `respond_to` methods to delegate which response to send back to the browser. So now whenever you hit the endpoint in a regular browser you would be served HTML, but when you explicitly request `JSON`, you get `JSON` back.

```ruby
# users_controller.rb

def show
  respond_to do |format|
    format.html
    format.json { render json: @user }
  end
end

def index
  @users = User.all
  respond_to do |format|
    format.html
    format.json { render json: @users }
  end
end
```

Now that my endpoints were setup, all that was left to do was use `ajax` to make requests to the endpoints, convert the `JSON` response to JavaScript Object prototypes, and then use JQuery inside the callback functions to render the data onto the page!

# [](#header-2)JavaScript/JQuery

I had originally planned on putting all of my JavaScript inside the JS files inside of the pipeline, but I quickly realized I needed to put the jQuery selectors on the html page itself to make sure that all of the correct elements were loaded on the page before the JavaScript loaded.

To render a resource index, I chose to render the user index of each trail, in order to display the hikers of each trail underneath the trail div. This part was pretty simple because I already had my `:trails` resource rendering all of its `:users`, I just had to convert each response to a JavaScript Object, and use its properties and methods to display the data on the page.

```javascript
function attachShowUsers() {
  $('.show-users-btn').click(function() {
    var id = $(this).data("id");
    var json = "/trails/" + id + ".json"
    $.get(json)
    .done(function(trailJSON) {
      // creating new Trail JS Object
      var trail = $.extend(new Trail(), trailJSON);

      var trailUsersId = "#trail-users-" + trail.id;
      var showUsersBtn = "#show-users-btn-" + trail.id;
      if ($(trailUsersId).children().length == 0) {
        // if users div is empty,
        $(showUsersBtn).text("Hide Hyykers");
        if (trail.users.length) {
          var html = '<h3 class="text-center">Hyykers</h3>';
          trail.users.forEach(function(userJSON) {
            var user = $.extend(new User(), userJSON);
            if (!html.includes(user.displayNameOrEmail())) {
              html += '<div class="trail-user-box">';
              html += '<h4>' + user.displayNameOrEmail() + '</h4>';
              html += '<a href="' + user.displayUrlPath() + '" class="btn btn-success trail-user">View Profile</a>';
              html += '</div>';
            }
          });
        } else {
          html = '<h3 class="text-center">No Hyykers Yet!</h3>';
        }
        $(trailUsersId).html(html);
      } else {
        $(trailUsersId).empty();
        $(showUsersBtn).text("Show Hyykers");
      }
    });
  });
}
```

I had a little trouble deploying to Heroku by using ES6 template literals when I rendered the data, so I had to botch it up a bit and use plain ol' string concatenation for now. But now I realize the utility of something like Handlebars.js, and I'll be refactoring it in the app really soon. 😉

For my show page I added in navigational buttons to navigate between `:trails`. When clicked, the navigation button renders a `GET` request for the next (or previous) `trail`, and renders its `data` along with its `users` and `comments`.

I had a few interesting problems to solve with this part of development. First off, I thought getting the next sequential Trail ID would be as easy as making a GET request to `"trails/" + trails.id + 1`. It worked when I created a few trails, but the problem was when I deleted a Trail (with an ID of 3), now whenever I was at Trail 2, the code would make a `GET` request for Trail 3 which didn't exist.
I tried using a recursive function, but that just made multiple `GET` requests when jumping from Trail 4 to Trail 7, if 5 and 6 were deleted.

I ended up created a variable to `GET` all trails, and then mapped that array of trails, into an array of their IDs. I was able to iterate over each ID number in the array in order to make the correct `GET` requests. Another neat bonus was being able to use that arrays length to avoid going past the last Trail when navigating with the nav buttons.

```javascript
  var lastTrailIndex = 0;
  var trailsIdArray = null;
  var currentIndex = null;
  var currentTrailId = $('#next-trail-btn').data("id");

  $(function() {
    attachNextButton();
    attachPrevButton();

    // Get lastTrail ID so you wont load past the next trail
    $.get('/trails.json')
    .done(function(trails) {
      trailsIdArray = trails.map(function(trail) {
        return trail['id'];
      });
      lastTrailIndex = trails.length - 1;
      currentIndex = trailsIdArray.indexOf(currentTrailId);
    });
  });
```

Rendering the `trails` data on the page was probably the most tedious part of the whole project. Each "Log Hike" button was a link with its own URL to create a new `Hike`, and each `Trail` had its own comments and users to render.

```javascript
function getPrevTrail(id) {
    var trailJsonUrl = "/trails/" + id + ".json";
    $.get(trailJsonUrl)
    .done(function(trailJSON) {
      var trail = $.extend(new Trail(), trailJSON);
      displayTrail(trail);
    });
  }

  function getNextTrail(id) {
    var trailJsonUrl = "/trails/" + id + ".json";
    $.get(trailJsonUrl)
    .done(function(trailJSON) {
      var trail = $.extend(new Trail(), trailJSON);
      displayTrail(trail);
    });
  }

  function displayTrail(trail) {
    // putting data into variables because heroku doesnt like template literals
    var distanceText = trail.distance + " mi."
    var elevationText = trail.elevation + " ft"
    var logHikeBtn = '<a href="/trails/' + trail.id + '/hikes/new" class="btn btn-warning">Log Hike</a>';
    $('#trail-distance').text(distanceText);
    $('#trail-elevation').text(elevationText);
    $('#trail-name').text(trail.name);
    $('.trail-name-span').text(trail.name);
    $('#trail-location').text(trail.location);
    $('#trail-type').text(trail.trail_type);
    $('#next-trail-btn').attr("data-id", trail.id);
    $('#prev-trail-btn').attr("data-id", trail.id);
    $('#trail-users').html(displayHikers(trail));
    $('#log-hike-btn').html(logHikeBtn);
    $('#trail-comments').html(displayComments(trail));
    $('.comment-form').hide();
  }

  function displayComments(trail) {
    var html = '';
    if (trail.comments.length > 0) {
      trail.comments.forEach(function(commentJSON) {
        var comment = $.extend(new Comment(), commentJSON);
        html += '<div class="trail-comment">';
        html += '<h4>' + comment.content + '</h4>';
        html += '<a href="/users/' + comment.userId + '">Hyyker</a>';
        html += '</div>'
      });
    } else {
        html += '<h2 id="no-hikers" class="text-center">No comments yet!</h2>';
    }
    return html;
  }

  function displayHikers(trail) {
    var html = "";
    if (trail.users.length > 0) {
      trail.users.forEach(function(userJSON) {
        var user = $.extend(new User(), userJSON);
        if (!html.includes(user.displayNameOrEmail())) {
          html += '<div class="hyyker text-center">';
          html += '<div class="hyyker-img">';
          html += '<img src="http://placekitten.com/g/125/125" width="80" height="80">';
          html += '</div>';
          html += '<div class="hyyker-email">';
          html += '<h3><a href="${user.displayUrlPath()}">' + user.displayNameOrEmail() + '</a></h3>';
          html += '</div>';
          html += '</div>';
        }
      });
    } else {
      html += '<h2 id="no-hikers" class="text-center">Looks like no Hyykers have hiked this trail yet!</h2>';
    }
    return html;
  }
```

For the last requirement, I was tasked to use a Rails form and AJAX to POST a new resource and render it to the page. I decided to add comment functionality on the Trails resources. First I added a nested `:comments` resource in the `:trails` resource

```ruby
resources :trails do
  resources :hikes
  resources :users, only: [:show]
  resources :comments, only: [:create, :show, :destroy]
end
```
Then made the Rails `form_for` a nested resource, and included `hidden_field` tags:

```ruby
<%= form_for([@trail, @comment]) do |f| %>
    <%= f.hidden_field :trail_id, value: @trail.id %>
    <%= f.hidden_field :user_id, value: current_user.id %>
    <%= f.hidden_field :user_email, value: current_user.email %>
    <%= f.text_area :content, class: "form-control" %><br>
    <%= f.submit "Post Comment", class: "btn btn-success" %>
  <% end %>
```

Next up was handling the form request with JavaScript by hijacking the form on submit, `POST`ing the data, and using JQuery to render the comments to the `#trail-comments` div.

```javascript
$('form').submit(function(e){
      e.preventDefault();
      var trailId = $('#comment_trail_id').val();
      var values = $(this).serialize();
      var posting = $.post("/trails/" + trailId + "/comments", values);
      posting.done(function(data){
        var comment = $.extend(new Comment(), data);
        console.log(comment.createdAt)
        var html = '<div class="trail-comment">';
        html += '<h4>' + comment.content + '</h4>';
        html += '<p>You just commented!</p>';
        html += '</div>';
        if ($('#no-hikers').length) {
          $('#trail-comments').html(html);
        } else {
          $('#trail-comments').append(html);
        }
      });
    });
```

Being able to request and render data asynchronously has opened my mind to so many possibilities, and definitely has me ready for learning React. Having a framework do all the heavy lifting of rendering views with javascript is a very welcome change and I can't wait to get deep enough in it to start making some serious projects!
