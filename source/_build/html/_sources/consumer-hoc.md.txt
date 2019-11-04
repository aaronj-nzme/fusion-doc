Using the Consumer Higher-Order Function
========================================

We created our first Feature and added it to the page - the only problem was, the content inside our component was hardcoded so it would always show the details for a single movie, Jurassic Park. Obviously the point of the component is to dynamically show movie details for _any_ movie, based on the user's request. This guide will walk through how to fix our components to use dynamic data provided by Fusion instead of static data.

Setting up a resolver
---------------------

A page in Pagebuilder is just that - it's just a statically served site that presents itself as how we have defined it on the Editor. While each feature on the page might fetch its own data for rendering and presenting the data, it isn't really suitable for receiving dynamic "global" content across the features.

For this purpose, we can leverage the use of templates and resolvers.

On the Template tab of the Pagebuilder page, you can create new templates. Let's create a new one and just name this `movies`. Once it's created, click on its name to go to the editor. Once at the editor, let's set this page up as you did on your first page you've created on the editor. On the Layout of your choice, add Movie Detail feature and let it show Jurassic Park. Afterwards, we will need to save this template - click on the `preview commit` in the Workflow section and then `Stage and Commit`.

Right now, this template will be blank and not do anything by itself. Don't fret! Now we will be using resolvers to populate this template with global content.

Let's go to the `Developer Tools` tab back in the Pagebuilder page, and `Resolvers` tab under it. Click on `new resolver` button to create your new resolver by filling out the fields as such:

    Name: movie-find
    Pattern: ^/movies/([\w\-]+)
    Priority: 1
    Template: movies
    Content Source: movie-find
    - MovieTitle: From Pattern: 1
    
    

Let's go more in-depth about what this does. As you might have noticed, the resolver relies on regex for matching the provided pattern. Essentially, this allows any URI in your page to be matched with the given pattern and be rendered by the designated template with the data provided by the Content Source. For example, if you have a URI `http://localhost/pf/movies/jurassic_park`, the above resolver will match with the `movies/jurassic_park` with the given Pattern `^/movies/([\w\-]+)`, and pass `jurassic_park` as the MovieTitle for the `movie-find` Content Source. the `movie-find` source will then use the MovieTitle param to fetch the data for the page as a global content.

Now that our resolver is fetching "global" content for us, let's see how we can use it in our component.

What is the Consumer function?
------------------------------

In Fusion, the Consumer higher-order function is what provides us dynamic data about the site and page the user requested, the outputType and layouts (if any) that are being used, any "global" content on the page, and more. Under the hood, `Consumer` is a higher-order function that wraps your components with `props` and instance methods that it can use to perform logic and render content.

It's not required for all features to be wrapped with `Consumer` if they don't require the data the Consumer provides - however, most of the time you'll need to since it's rare to have entirely static Feature components.

Adapting our Feature
--------------------

In this example, we want to access the title, director, list of actors, and more info associated with the movie the user requested. Now that we've fetched that content in our resolver, it should be available as `this.props.globalContent` provided by the Consumer. Let's wrap our component with the `Consumer` decorator and see what changes:

    /*  /components/features/movies/movie-detail.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class MovieDetail extends Component {
    
      {/* The `constructor` and `togglePlot` methods remain unchanged */}
      ...
    
      render () {
    
        {/* The `isPlotShown` and `plotButton` constants remain unchanged */}
        ...
    
        {/* Here, we extract the data we want from `this.props.globalContent`, which we "short circuit" default to an empty object, just in case it doesn't exist */}
        const { Actors, Director, Plot, Poster, Rated, Title, Writer, Year } = this.props.globalContent || {}
    
        {/* Replace the static values with their dynamic equivalents, first checking if each necessary value exists */}
        return (
          <div className='movie-detail col-sm-12 col-md-8'>
            {Title && <h1>{Title}</h1>}
            {Director && <p><strong>Director:</strong> {Director}</p>}
            {Actors && <p><strong>Actors:</strong> {Actors}</p>}
            {Plot && <p><strong>Plot:</strong> {isPlotShown && Plot} {plotButton}</p>}
            {Rated && <p><strong>Rated:</strong> {Rated}</p>}
            {Writer && <p><strong>Writer:</strong> {Writer}</p>}
            {Year && <p><strong>Year:</strong> {Year}</p>}
            {Poster && Title && <img src={Poster} alt={`Poster for ${Title}`} />}
          </div>
        )
      }
    }
    
    export default MovieDetail
    

A few things have changed about our component:

*   We're now importing the `Consumer` object from `fusion:consumer` at the top of the file
*   On the line above our class definition we've added the `Consumer` [decorator](https://www.sitepoint.com/javascript-decorators-what-they-are/)
*   Inside our `render` method we're extracting the pieces of data we need from `this.props.globalContent` as constants.
*   We then check whether each piece of data exists, and if so render the appropriate piece of markup.

Now we have our component _and_ content source defined, and our resolver fetching content on page load - the last part is to add the Feature to a template and see it in action.

And just like that, our component is rendering content dynamically! Go ahead and publish the page and try requesting the URL defined in our resolver with different movie names to see it fetch different content.

> **NOTE**
> 
> It's possible to wrap a [functional component](https://reactjs.org/docs/components-and-props.html#functional-and-class-components) in the `Consumer` higher-order function and still get props passed as in the class-based syntax - however, only the class-based syntax allows you to use `Consumer`'s instance methods.