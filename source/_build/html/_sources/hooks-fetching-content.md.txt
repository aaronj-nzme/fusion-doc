Fetching Content
================

> NOTE  
>   
> The main difference between the class component and the functional component is that the pagination is not yet supported, as noted at the bottom of the page. This is being worked on and will be updated in the future.

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
    import Content, { useContent } from 'fusion:content';
    import React, { Component } from 'react';
    
    const MovieList = (props) => {
      const movies = [];
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
    
    export default MovieList
    

Right now, our component doesn't do much - we are initializing an empty `movies` array we are supposed to call the map function on, but the loop just outputs an empty `<div>` right now. So we need to 1) fetch some movies data and 2) output some content inside our movies loop. Let's call a method to do the fetching:

    ...
      const movies = useContent({
        source: 'movie-search', 
        query: { movieQuery: 'Jurassic' }, 
        filter: '{ totalResults Search { Title Year Poster } }',
        transform (data) {
          // Check if data is being returned
          if(data && data.Search)
            return { list: [...data.Search] };
    
          // Otherwise just keep the current list of movies
          else
            return movies;
        }
      })
    ...
    

Here, we're utilizing Fusion's `useContent` hook to fetch some content and then set some state. The function takes in one object variable key we call `contentConfigMap`, with the key that you would like to use to retrieve in the state.

For our purpose, we will be putting three parameters in the `contentConfigMap` object:

*   `source`: The name of the content source (`movie-search` for now)
*   `query`: Object that contains the values we actually want to query on - in this case, a `movieQuery` param searching for movies with the word 'Jurassic' in them (this object will be the only argument passed to the `resolve` method in our content source).
*   `filter`: A string containing a GraphQL query object, which will filter the results of our JSON down to just the data we need for this component
*   `transform`: the function that will destructure the result of the query to cleanly insert them into the state.

At this point our fetch should be working! The last problem is we aren't displaying any data. Let's fix that too:

      ...
      const {list: movieList = []} = movies;
    
      return (
        <Fragment>
          <h2>Movies</h2>
          <div>
            {movieList && movieList.map((movie, idx) =>
              <div key={`movie-${idx}`}>
                <h4>{movie.Title}</h4>
                <p><strong>Year:</strong> {movie.Year}</p>
                <img src={movie.Poster} />
              </div>
            )}
          </div>
        </Fragment>
      )
    
    

This kind of component would formerly be known as `stateless` component. However, hooks allow these components to use React features, and that's what useContent will do - it will allow you to add React state to this "stateless" component. In fact, React would actually prefer you call this simply a [functional component](https://reactjs.org/docs/hooks-state.html).

We can now simply iterate over the `movies` array in our state and output the information we want (`Title`, `Year`, `Poster`) for each movie as if they'd always been there. This should result in a working component that fetches and displays data about movies with the word 'Jurassic' in the title! Let's see the entire component together:

    /*  /components/features/movies/movie-list.jsx  */
    import Content, { useContent } from 'fusion:content';
    import React, { Fragment, Component } from 'react';
    const MovieList = (props) => {
      const movies = useContent({
        source: 'movie-search', 
        query: { movieQuery: 'Jurassic' }, 
        filter: '{ totalResults Search { Title Year Poster } }',
        transform (data) {
          // Check if data is being returned
          if(data && data.Search)
            return { list: [...data.Search] };
    
          // Otherwise just keep the current list of movies
          else
            return movies;
        }
      })
    
      const {list: movieList = []} = movies;
    
      return (
        <Fragment>
          <h2>Movies</h2>
          <div>
            {movieList && movieList.map((movie, idx) =>
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
    
    export default MovieList
    

Adding pagination
-----------------

The current method of rendering with hooks does not support pagination at the moment. This is being investigated on and will be updated in the future - stay tuned!