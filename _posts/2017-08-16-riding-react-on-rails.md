---
published: true
layout: post
title: "Building a React App With A Rails Backend!"
author: "Sergio"
permalink: /2017/08/16/riding-react-on-rails
date: '2017-08-16 16:16:01 -0600'
---

Coming up with a proper project idea to demonstrate my knowledge was pretty tough. I had so many ideas I wanted to implement, but for my final project I thought it would be sweet to pay a final homage of sorts to my current (soon-to-be-ex) career of bartending. I wanted to create something useful, and I wanted to be able to use at least [one](https://developers.google.com/places/) (ended up using [two](https://unsplash.com/developers)!) API of some sort.

Through this project I was able to solidify a ton of concepts, and use some awesome new technologies, React and Redux.

I decided to use the [Google Places API](https://developers.google.com/places/) to gather information on the users current location, and display results for nearby Bars, Restaurants, and Dives to help users find their next spot to "turn up" at. ;)

Simple? Of course. But I do plan on adding some algorithms to calculate the distance of the bar divided by the average prices of drinks at the establishment in order to find the best bang-for-your-buck place to drink. *Unfortunately,* I didn't find an API specific enough to find the prices of menu items. And probably for good reason, drink prices can be as variable as gas prices in some places. But perhaps it would make an awesome project for the future.

## Planning

Before getting my hands dirty, I broke the project up into small, manageable tasks to get everything set up and running:

	- Set Up Rails API
	- Generate model for "Bars"
	- Set up React App (made easy using 'create-react-app')
	- Create NavBar, Main, BarsList, BarsPage, and BarsShow components
	- Find a way to get users current location and pass that as data to the Google Places API

These were my initial goals of the project *just* to get everything started. I knew I wanted to implement [Redux](http://redux.js.org/) eventually, but not until I got the initial complexities set up. We'll get into that later.

# The Railsy Stuff

The rails side of the project went amazingly smooth. I knew how to implement everything like migration files and controllers manually, but setting everything up was as easy as running `rails g scaffold Bars` and Rails magic prevailed and set up all the files I needed. I gave my bars simple properties like `name`, `address`, and `rating`.

For fetching business information, I used a nifty little gem called [google_places](https://github.com/qpowell/google_places). This gem provided a Ruby wrapper around the Google Places API. I created a separate controller route: google_places, that would receive requests from the frontend.

This worked out great. Any external API requests were made from my Rails server, and if I ever needed that information on the frontend I could just dispatch an action to Redux and have it fetch that data. Neat!

# The React Stuff

By using [create-react-app](https://github.com/facebookincubator/create-react-app), I was able to bootstrap most of my app with a single command. All that was needed was to start creating presentational components. For the sake of adding in minimal styling later, I ended up using the `material-ui` [npm-module](https://www.npmjs.com/package/material-ui) to import Material Design components into my app for a clean look.

I created my navbar as a `material-ui` `AppBar`, and added cool-looking links in my sidebar (`BarsList`) in order to display the bars fetched from the Google Places API. I created a `Main` component that would be in charge of using [React Router](https://github.com/ReactTraining/react-router) to switch out `Routes` of presentational components.

In order to get the location of the users, I used the browser's built-in `navigator` API and used its `getLocation()` method to get the `latitude` and `longitude` coordinates. I used those coordinates and sent them as a `POST` request to my `api/google_places` endpoint, which in turn made a `POST` request to the Google Places API and queried for bars around those coordinates. That data was then sent to my BarsList component which was in charge of rendering links for each Bar Route.

My BarsShow component does most of the heavy lifting for each Bar. Its in charge of fetching a new photo from the Unsplash API (fetching photos from Google places was difficult without making a ton of requests to the API). Due to the nature of React Router, whenever a new bar was clicked and a new Route was rendered, the image would stay the same. I came up with a solution to compare the current barId params of the Route URL with the previous params with `componentDidUpdate()` and fetch a new photo whenever there was a difference.

```javascript
  class BarsShow extends Component {
  componentDidMount() {
    this.props.fetchPhoto()
  }

  componentDidUpdate(prevProps) {
    if (this.props.match.params.barId !== prevProps.match.params.barId) {
  this.props.fetchPhoto()
    }
  }
...
```

Google places provided a number rating for each business, but I wanted to implement a graphical rating. I used the rating number to `parseInt()` and translate that number into however many FontAwesome star components I wanted to display.

```javascript
  renderStars = (rating) => {
    let numOfStars = parseInt(rating)
    let faIcons = []
    for (var i = 0; i < numOfStars; i++) {
      faIcons.push(<FontAwesome name="star" />)
    }
    return faIcons
  }

```

The real magic of the component comes alive when I had to figure out which bar to display, based on the route params:

```
const mapStateToProps = (state, ownProps) => {
  const bar = state.bars.find(bar => bar.id === ownProps.match.params.barId)
  const comments = state.comments.filter(comment => comment.barId === ownProps.match.params.barId)

  if (bar) {
    return {
      bar: bar,
      comments: comments,
      photo: state.photo
    }
  } else {
    return { bar: {} }
  }
}
```

Using Redux, I was able to keep all of the bar data in the state, and filter out the unneeded data based on the routes :barId params for whatever bar I wanted to display.

Now would probably be a good time to talk about how I inserted Redux. After realizing my app was starting to get more complex, I knew I was going to need to access my app state across more of my components.

Redux is awesome because I can keep my state all in one object, and query that object for that specific piece of state whenever I needed it.

In order to delegate state, I created a few reducers, and imported them into my rootReducer which delegated the different pieces of state:

```javascript
export default combineReducers({
  bars: barsReducer,
  comments: commentsReducer,
  photo: photoReducer
})
```
My main reducer being the barsReducer:

```javascript
export default (state = [], action) => {
  switch (action.type) {
    case 'FETCH_BARS':
      return state.concat(action.payload)
    case 'ADD_BAR':
      const bar = Object.assign({}, action.payload, { id: state.length + 1 })
      return state.concat(bar)
    default:
      return state
  }
}
```

This basically means, whenever I dispatch an action through an action creator, like 'ADD_BAR', create a new object with the new bar, and concat that onto the current state and return a new array containing all of the bars in my state, along with the new bar I just added.

Here are a couple of my actionCreator functions for my bar resource:

```javascript
export function fetchBars() {
  return (dispatch) => {
    dispatch({ type: 'LOADING_BARS' })
    return fetch(`${API_URL}/bars`)
      .then(response => response.json())
      .then(bars => dispatch({ type: 'FETCH_BARS', payload: bars }))
  }
}

export function addBar(bar) {
  return {
    type: 'ADD_BAR',
    payload: bar
  }
}
```

As you can, fetchBars() simply fetches the list of bars in my bars API, and addBar() dispatches an ADD_BAR action and passes along a payload of the newly created bar.

With Redux, my actions and reducers were able to work together to provide the data I needed throughout my application, and helped my components be more modular and free of state management. I can't wait to work on bigger projects to really reap the benefits of Redux as my projects become more complex.

All in all I had an awesome time building this application, and I already have a ton of ideas to expand upon it:

  - Using Google Maps API to display maps of businesses
  - Perhaps using another API besides Google Places in order to get relevant photos. As beautiful as Unsplash photos are, I would like to see the actual bar photos. :)
  - Maybe even use the Twilio API to text friends which bar is going to be the meetup spot along with a link for directions

This was a dope learning experience, and I can't wait to see whats next. :)
