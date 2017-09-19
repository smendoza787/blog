---
published: true
layout: post
title: "Building a RESTful API with Node and Express.js"
author: "Sergio"
permalink: /2017/09/16/node-express-pets-api
date: '2017-09-16 16:16:01 -0600'
---

Since I've graduated the full stack web developer program, I can't help but have a wandering eye towards all the new and latest technologies. So far, I am most proficient in Ruby on Rails. I can get a quick API up and running, and I can even take advantage of Rails views to deliver a full stack app with Rails alone. But once I started using React on the front end of my projects, I realized how annoyingly slow it was to refresh and redirect the page to so many different routes. Which led me to doing more research on how to build applications using JavaScript throughout the entirety of an application. I learned that having JavaScript handle all of your front AND back end, your application could experience some major speed improvements! Okay enough talking, lets jump into the project.

In this project we're going to create a simple RESTful API for pets, because who doesn't love doing another pet project amirite?

If you'd like to refer to the finished project you can see it at https://github.com/smendoza787/node-express-pet-api

Here are a list of tools we'll be using throughout:

- Node.js
- MongoDB
- [Postman](https://www.getpostman.com/)

This post is going to assume you have Node and MongoDB installed on your machine. If you are unsure, you can type `node -v` and `mongo --version` in your terminal and make sure you have at least some version of them installed, if not you can click [here for Node](https://nodejs.org/en/download/) or [here for MongoDB](https://docs.mongodb.com/getting-started/shell/installation/) and follow instructions for your platform.

# Getting Started

1. Lets create a directory for our project by running `mkdir nodePetsApi` in our terminal
2. Navigate into our newly created directory with `cd nodePetsApi`
3. create a new `package.json` by running `npm init`
    - The `package.json` file contains all of the metadata for our node application. More importantly, it holds all of the dependencies for our app. If you're familiar with Ruby, its similar to a Gemfile.
    - `npm init` will ask you for a series of inputs regarding the application such as the name, version, and a description of the application. Fill these out and hit enter when it prompts you if everything looks ok.
    - When you're done you should have something like this:
    ![Imgur](https://i.imgur.com/m8WTcjz.png)
    You can go ahead and hit enter to create your package.json
4. Create a `server.js` file by running `touch server.js`
    - This file will contain the protocol for running our server
5. Create a folder named `api` - `mkdir api`
    - Inside this folder we'll create 3 subfolders named `controllers models routes`
    - `mkdir api/controllers api/models api/routes`
6. Create `petController.js` in the `controllers` folder, `petRoutes.js` in the `routes` folder, and `petModel.js` in the `model` folder
    - `touch api/controllers/petController.js api/routes/petRoutes.js api/models/petModel.js`

So far, our file structure should look like this:

![Imgur](https://i.imgur.com/6B9q8dm.png)

# Server Setup

Now we can install express and nodemon. Express will be used to create our server, while nodemon will help us keep track of changes we make to our files, and automatically restart the server on every save.

`npm install --save-dev nodemon`

`npm install --save express`

This is going to add nodemon and express to our dependencies in our package.json.

1. Open package.json and add this task to our `scripts` section:
    - `"start": "nodemon server.js"`

![Imgur](https://i.imgur.com/lPRrSze.png)

2. Open server.js and type in the following:

```javascript
  var express = require('express'),
      app = express(),
      port = process.env.PORT || 3000;

  app.listen(port);

  console.log('pets API server started on: ' + port);
```

3. In your terminal, run `npm run start`. This will start the server, and you should see:

`pets API server started on: 3000`

# Setting up the schema

In order to setup our schema, we're going to use `mongoose`, a Javascript library that will serve as an Object-Document Model (ODM) made specifically for MongoDB. The Schema that we create is going to serve as a blueprint for all of our future pet instances.

Install mongoose by running the following on your terminal:

`npm install --save mongoose`

Once installed, type the following into `api/models/petModel.js`:

```javascript
  var mongoose = require('mongoose');
  var Schema = mongoose.Schema;


  var PetSchema = new Schema({
    name: {
      type: String,
      required: 'Please enter the name of your pet'
    },
    type: {
      type: String,
      required: 'Please enter the type of your pet'
    },
    mood: {
      type: [{
        type: String,
        enum: ['hungry', 'happy', 'sleepy']
      }],
      default: ['happy']
    },
    Created_date: {
      type: Date,
      default: Date.now
    }
  });

  module.exports = mongoose.model('Pet', PetSchema);
```

In this file, we required mongoose and its Schema function in order to implement our Schema. Then we created a PetSchema, a new instance of mongoose.Schema, and instantiated it with an object whose properties will reflect one of our Pet models.

Our pet Schema will consist of a `name` and a `type`, both of which are required Strings. It will also consist of a `mood` which enumerates between 3 different moods, but defaults to `happy` when first created. Lastly it will contain a Created_date which defaults to the time the document was created.

Finally, in order to use our schema, we need to convert our `PetSchema` into a Model we can work with. We can do this by passing our schema into `mongoose.model(modelName, schema)`.

# Setting up our routes

Routing refers to determining how an application responds to a client request to a particular endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, and so on).

Each route can have one or more handler functions, which are executed when the route is matched.

In your `api/routes/petRoutes.js` file, please type the following:

```javascript
  module.exports = function(app) {
    var pets = require('../controllers/petController');

    // pet Routes
    app.route('/pets')
      .get(pets.list_all_pets)
      .post(pets.create_a_pet);


    app.route('/pets/:petId')
      .get(pets.read_a_pet)
      .put(pets.update_a_pet)
      .delete(pets.delete_a_pet);
  }
```

We have defined two basic routes: `/pets` and `/pets/:petId` with different HTTP methods for each.

- GET `/pets`
- POST `/pets`
- GET `/pets/:petId`
- PUT `/pets/:petId`
- DELETE `/pets/:petId`

We've also required the petController so each route has access to its own route handler function. The route handler function gets executed when the route is matched.

# Setting up the controller

We'll be defining the 5 route handler functions we mentioned in our `petRoutes.js` file. We'll be using different mongoose methods to interact with the database, and we'll be exporting each of these functions in order to `require` them in different parts of our app.

Open `api/controllers/petController.js` and type the following:

```javascript
  var mongoose = require('mongoose'),
      Pet = mongoose.model('Pet');

  exports.list_all_pets = function(req, res) {
    Pet.find({}, function(err, pet) {
      if (err) {
        res.send(err);
      }
      res.json(pet);
    });
  };

  exports.create_a_pet = function(req, res) {
    var new_pet = new Pet(req.body);
    new_pet.save(function(err, pet) {
      if (err) {
        res.send(err);
      }
      res.json(pet);
    });
  };

  exports.read_a_pet = function(req, res) {
    Pet.findById(req.params.petId, function(err, pet) {
      if (err) {
        res.send(err);
      }
      res.json(pet);
    });
  };

  exports.update_a_pet = function(req, res) {
    Pet.findOneAndUpdate({_id: req.params.taskId}, req.body, {new: true}, function(err, pet) {
      if (err) {
        res.send(err);
      }
      res.json(pet);
    });
  };

  exports.delete_a_pet = function(req, res) {
    Task.remove({
      _id: req.params.petId
    }, function(err, pet) {
      if (err) {
        res.send(err);
      }
      res.json({ message: 'Pet successfully deleted' });
    });
  };
```

# Putting it all together

Earlier, we set up our `server.js` file with minimal code to get it up and running. Now we'll connect our handlers(controllers), database, models, and routes together.

In order to use `body-parser`, we'll have to install it with npm first:

`npm install --save body-parser`

Make sure your `server.js` file looks like the following, afterwards we'll go over the changes we made:

```javascript
  var express = require('express'),
      app = express(),
      port = process.env.PORT || 3000,
      mongoose = require('mongoose'),
      Pet = require('./api/models/petModel'),
      bodyParser = require('body-parser');

  // mongoose instance connection url connection
  mongoose.Promise = global.Promise;
  mongoose.connect('mongodb://localhost/Petdb', { useMongoClient: true });

  app.use(bodyParser.urlencoded({ extended: true }));
  app.use(bodyParser.json());

  var routes = require('./api/routes/petRoutes'); //importing route
  routes(app); //register the route

  app.listen(port);

  console.log('pets API server started on: ' + port);
```

- First we loaded up mongoose in order to connect it to our mongoDB database that will run on `localhost/Petdb`.

- Then we required our Pet model that we set up in `api/models/petModel`.

- Next we required body-parser in order to parse incoming requests. Then we applied it as middleware to our app by using `app.use()` before registering our routes.
- Finally we required the routes function that holds all of our route handler functions, and executed that function while passing in our `app`

# Starting up our MongoDB database

1. Open up a new terminal (cmd + T on Mac), and run `mongod` in  your terminal
  - `mongod` is the Mongo Daemon, it is essentially the host process for the database. It essentially says, start the MongoDB process and run it in the background.
2. Open up _another_ terminal and run `mongo`, this opens up the Mongo shell in order to connect to an instance of MongoDB. In the prompt, type `use Petdb` which will create a new database collection named `Petdb`

Now go to your browser and visit `http://localhost:3000/pets`. If you see an empty array `[]` that means we're up and running! We only see an empty array because we haven't added anything to the database. Lets do that now with Postman.

# Testing with Postman

Now that everything is connected, we can use a simple tool called Postman in order to send requests to our API and pass along any parameters we need to create a new Pet instance.

1. Open up Postman, change the method to 'POST', and enter `http://localhost:3000/pets` in the URL field
2. Click the 'body' tab and select 'x-www-form-urlencoded'
3. In the first key field, type in `name` and give your pet a name
4. In the second key field, type in `type` and state what type of pet you're entering
4. Click Send

We should get a response with the JSON object of our newly created pet!

![Imgur](https://i.imgur.com/mNJMJCV.png)

Notice how our pet was created with a default mood of `happy`, but we could have created it with any of our other moods as well.

![Imgur](https://i.imgur.com/Zm9wuSA.png)

Now when we send a GET request to `http://localhost/pets` we should get a list of our created pets.

And thats it! I hope you learned something from reading this tutorial, as much as I learned from making it! :)
