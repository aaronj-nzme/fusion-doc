Isomorphic vs. Server vs. SPA rendering
=======================================

One of the enormous benefits of writing Fusion Feature Packs as React components is the ability to render them ["isomorphically"](https://en.wikipedia.org/wiki/Isomorphic_JavaScript) \- meaning on both the server-side and again on the client-side. However, isomorphic rendering isn't the only option when writing Features - Fusion gives you the flexibility to choose what context you want each of your Features to render in.

For Fusion's purposes, Output Types will always be rendered server-side only, since they are the HTML shell of the page itself. Layouts and Chains will always be rendered isomorphically, since they may contain multiple Features - some that are server-side only, and some client-side only. Features, however, can be configured to render isomorphically, server-side only, or client-side only. Let's talk about why you'd want to use each rendering option, and how to do so.

Rendering isomorphically/universally
------------------------------------

Isomorphic rendering offers lots of benefits when building a modern [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application) \- you get the rich interactivity and performance benefits of running a client-side web application, without losing SEO-optimization or the ability to render in non-JavaScript-enabled clients.

If a Feature you're building has some sort of client-side interaction (`onClick`, `onSubmit`, `keyPress` events, etc.), but you also need to render it server-side, you'll want to render isomorphically.

The good news is isomorphic rendering is the default in Fusion - if you'd like to render a Feature isomorphically, simply write your components normally and they'll render in both contexts.

Rendering server-side only
--------------------------

Sometimes, isomorphic rendering won't be appropriate for a certain component. Perhaps you're writing a component that uses some code that shouldn't be exposed to the client, or maybe you just don't want to send the extra bytes of JavaScript to the browser for this component since it never gets updated client-side. For these situations, you may want to render server-side only.

If you'd like to mark a Feature to be rendered on the server only, you can add a `.static = true` property to it like so:

    /*  /components/features/movies/movie-detail.jsx  */
    
    import React, { Component } from 'react'
    
    class MovieDetail extends Component {
      ...
    }
    
    MovieDetail.static = true
    
    export default MovieDetail
    

This will tell Fusion that this component should be rendered into HTML on the server-side, but then **not** re-hydrated as a React component client-side.

**WARNING**

There are downsides to rendering server-side only. Firstly, your component will be wrapped in a containing `div` that tells Fusion not to reload it client-side - this could affect your styling. Another downside is that your content may become stale if it changes before Fusion's CDN has expired it.

Good candidates for static components meet the following criteria:

*   They have no client-side interactivity implemented by React or Fusion
*   They don't use any content from a content source at all, OR they use **only** `globalContent`, not feature-level content, as it's easier to purge caches for `globalContent` and therefore less likely to serve stale content

Rendering client-side only
--------------------------

The final option available to us is rendering client-side only. This is a bit of a misnomer with Fusion, as your React component's `render` function will get invoked on the server no matter what. However, you can choose to render empty markup (without content) on the server, and then fill in the component with content on the client-side later. This may offer a small performance benefit, as you'll avoid fetching content for this component server-side. This option is best for Features that are not crucial to the content of the page and can be loaded with a small delay.

In Fusion, there are two ways to ensure that code **only** gets run on the client side: use a conditional to check for the `window` object (which is only available on the client), or to put client-side code inside the `componentDidMount` React lifecycle method. This method will get triggered once your component gets mounted on the page client-side, but does not get executed server-side.

Let's see how that might look in our `MovieList` component:

    /*  /components/features/movies/movie-list/default.jsx  */
    import Consumer from 'fusion:consumer'
    import React, { Fragment, Component } from 'react'
    import './style.scss'
    
    @Consumer
    class MovieList extends Component {
      constructor (props) {
        super(props)
        this.state = { movies: [], page: 1 }
      }
    
      componentDidMount() {
        // We moved our `this.fetch()` call to `componentDidMount` from `constructor`
        this.fetch()
      }
    
      fetch () {
        ... // All our fetching logic from earlier is still here
      }
    
      render () {
        // If the window object doesn't exist, we will return an empty Fragment
        if (typeof window === 'undefined') return null
    
        ... // All our rendering logic for the rest of the component is still here
      }
    }
    
    MovieList.propTypes = {
      customFields: PropTypes.shape({
        movieListConfig: PropTypes.contentConfig('movies')
      })
    }
    
    export default MovieList
    

We've done two things to ensure our component doesn't fetch or render server side:

*   We moved our content fetch call inside of `componentDidMount`, so now it will only occur client-side
*   Additionally, we added a check in the `render` method to render an empty `Fragment` if the `window` object doesn't exist, which will only be true on the server.

The effect of both these changes is that the HTML rendered from our component on the server will be empty initially, and content will only be fetched and fill the component on the client-side!

**WARNING**

Rendering client-side only could affect SEO adversely, since web crawlers typically have difficulty reading AJAX-only web pages - although [they have been improving](https://developers.google.com/search/docs/ajax-crawling/docs/learn-more).