
Fetching Content
================

The whole point of defining a content source, a GraphQL schema, and putting our API credentials in environment variables was so we can use them to retrieve content - and now we're finally ready to do so!

Using "global" content vs. fetched content
------------------------------------------

It's important to remember that you only need to fetch content if the content you need has not already been provided to you by `globalContent`, or if you want to retrieve some content client-side. If the content you need in your feature is part of the "global" content of the page, meaning it is semantically identified by the URL the user requested, then you probably don't need to fetch at all.

A quick example: if you have a feature called `authors` whose purpose is to list the authors of the main story on the page, then you will want to use `props.globalContent` \- since the information in the `authors` component is semantically tied to the main story. If, however, you are building an unrelated `sports_scores` component that shows the most recent scores from local sports games, that content will almost certainly _not_ exist as part of your main story - so you'll need to fetch it separately.

> NOTE Even though you may not need to fetch "feature specific" content in your Feature Pack, you still need to define content sources so resolvers can use them to fetch "global" content. Without content sources you can't get "global" content or "feature specific" content.

Fetching content and setting state
----------------------------------

Once we've determined we need to fetch content, we need some more information:

*   when do we want to fetch the content (on the server only, the client only, or both?)
*   what content source do we want to fetch from?
*   what arguments do we need to pass to get the content we want?
*   what pieces of data from the returned content do we actually need?


Let's define a simple component called `MovieList` for this purpose:

    /*  /components/features/movies/movie-list.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class MovieList extends Component {
      constructor (props) {
        super(props)
        this.state = { movies: [] }
      }
    
      render () {
        const { movies } = this.state
        return (
          <div className='movie-list col-sm-12 col-md-4'>
            <h2>Movies</h2>
            <div className='movie row'>
              {movies && movies.map((movie, idx) =>
                <div className='col-sm-12 border' key={`movie-${idx}`}>
                  {/* display movie info here */}
                </div>
              )}
            </div>
          </div>
        )
      }
    }
    
    export default MovieList
    

Right now, our component doesn't do much - we are initializing an empty `movies` array in our `state` object that the `render` method loops over, but the loop just outputs an empty `<div>` right now. So we need to 1) fetch some movies and add them to our `state`, and 2) output some content inside our movies loop. Let's add a method to the class to do the fetching:

      fetch () {
        const { movies } = this.state;
        this.fetchContent({
            movies: {
              source: 'movie-search', 
              query: { movieQuery: 'Jurassic' }, 
              filter: '{ totalResults Search { Title Year Poster } }',
              transform (data) {
                // Check if data is being returned
                if(data && data.Search)
                  return { list: [...movies.list, ...data.Search] };
    
                // Otherwise just keep the current list of movies
                else
                  return movies
              }
            }
        })
      }
    

Here, we're utilizing Fusion's fetchContent method to fetch some content and then set some state. The function takes in one object variable key we call `contentConfigMap`, with the key that you would like to use to retrieve in the state.

For our purpose, we will be putting three parameters in the `contentConfigMap` object:

*   `source`: The name of the content source (`movie-search` for now)
*   `query`: Object that contains the values we actually want to query on - in this case, a `movieQuery` param searching for movies with the word 'Jurassic' in them (this object will be the only argument passed to the `resolve` method in our content source).
*   `filter`: A string containing a GraphQL query object, which will filter the results of our JSON down to just the data we need for this component
*   `transform`: the function that will destructure the result of the query to cleanly insert them into the state.

This method should work great - except we haven't invoked it anywhere! Let's change that in our `constructor` method:

      constructor (props) {
        super(props)
        this.state = { movies: { list: [] }}
        this.fetch = this.fetch.bind(this)
        this.fetch();
      }
    

> **NOTE**
> 
> Because we're invoking the `fetch` method in the `constructor`, our fetch will occur on both the server _and_ the client side when we're rendering. If we had just wanted to invoke client side, we could have put the `fetch` call inside of `componentDidMount` instead of the `constructor`, since `componentDidMount` only occurs client side.

At this point our fetch should be working! The last problem is we aren't displaying any data. Let's fix that too:

      render () {
        const { list: movies } = this.state.movies
        return (
          <Fragment>
            <h2>Movies</h2>
            <div>
              {movies && movies.map((movie, idx) =>
                <div key={`movie-${idx}`}>
                  <h4>{movie.Title}</h4>
                  <p><strong>Year:</strong> {movie.Year}</p>
                  <img src={movie.Poster} />
                </div>
              )}
            </div>
          </Fragment>
        )
      }
    

Because React will re-render automatically whenever there is a change to the `state` or `props` of our component, and we're triggering a state change when we fetch our new movies, we can simply iterate over the `movies` array in our state and output the information we want (`Title`, `Year`, `Poster`) for each movie as if they'd always been there. This should result in a working component that fetches and displays data about movies with the word 'Jurassic' in the title! Let's see the entire component together:

    /*  /components/features/movies/movie-list.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Fragment, Component } from 'react'
    
    @Consumer
    class MovieList extends Component {
      constructor (props) {
        super(props);
        this.state = { movies: { list: [] }};
        this.fetch = this.fetch.bind(this)
        this.fetch();
      }
    
      fetch () {
        const { movies } = this.state;
        this.fetchContent({
            movies: {
              source: 'movie-search', 
              query: { movieQuery: 'Jurassic' }, 
              filter: '{ totalResults Search { Title Year Poster } }',
              transform (data) {
                // Check if data is being returned
                if(data && data.Search)
                  return { list: [...movies.list, ...data.Search] }
    
                // Otherwise just keep the current list of movies
                else
                  return movies;
              }
            }
        })
      }
    
    
      render () {
        const { list: movies } = this.state.movies
        return (
          <Fragment>
            <h2>Movies</h2>
            <div>
              {movies && movies.map((movie, idx) =>
                <div key={`movie-${idx}`}>
                  <h4>{movie.Title}</h4>
                  <p><strong>Year:</strong> {movie.Year}</p>
                  <img src={movie.Poster} />
                </div>
              )}
            </div>
          </Fragment>
        )
      }
    }
    
    export default MovieList
    

Adding pagination
-----------------

Unfortunately, this only fetches the _first page_ of movies with "Jurassic" in the title from OMDB. But since OMDB's API allows us to send a `page` param, and our content source is already set up to accept such a param, it's easy to add pagination to this feature as such:

    /*  /components/features/movies/movie-list.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Fragment, Component } from 'react'
    
    @Consumer
    class MovieList extends Component {
      constructor (props) {
        super(props)
        this.state = {
          movies: {
            pages: []
          },
          page: 0
        }
        this.fetch = this.fetch.bind(this)
        this.fetch()
      }
    
      fetch () {
        const { page } = this.state;
    
        // Increment the page at each call
        this.state.page += 1;
    
        this.fetchContent({
          movies: {
            source: 'movie-search',
            query: {
              movieQuery: 'Jurassic',
              page: page + 1
            },
            filter: '{ totalResults Search { Title Year Poster } }',
            transform: (data) => {
              // Check if data is being returned
              if(data && data.Search) {
                // Add the results to the paginated list of movies
                this.state.movies.pages[page] = data.Search
                return this.state.movies
              }
    
              // Otherwise just keep the current list of movies
              else{
                return this.state.movies;
              }
            }
          }
        })
      }
    
      render () {
        // Concatenate the lists of the movies and filter duplicates - this would ensure that
        // a multiple clicks on the 'More' button wouldn't cause issues with incomplete and out-of-order fetches from
        // network issues
        const movies = [].concat(...this.state.movies.pages).filter(movie => movie);
    
        return (
          <Fragment>
            <h2>Movies</h2>
            <div>
              {movies && movies.map((movie, idx) =>
                <div key={`movie-${idx}`}>
                  <h4>{movie.Title}</h4>
                  <p><strong>Year:</strong> {movie.Year}</p>
                  <img src={movie.Poster} />
                </div>
              )}
              <button onClick={ this.fetch }>More</button>
            </div>
          </Fragment>
        )
      }
    }
    
    export default MovieList
    

With this, all we had to do to get pagination working was:

*   add a `page` property to our state object and initialize it to `0` \- this will be incremented at the first call to the `fetch` function.
*   increment the `page` whenever we fetch, so next time we'll fetch the following page - since it's not necessary to re-render the component, we can increment it directly.
*   include the page variable with 1 added to it (note that we are not passing the state's incremented page value) - this is so that we can preserve the ordering of the paginated list of the movies and make sure the list starts properly at the index of 0 when we assign the result of the fetch request to the list.
*   Add a button at the bottom of the component that allows us to call the `fetch` method, which should get the next page of results, display the new results, and increment the page all at once.

And that's how you fetch content in Fusion. Phew!