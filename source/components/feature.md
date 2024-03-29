Creating a Feature Component
============================

After Output Types, Features are probably the most important components in your Feature Pack - after all, that's where the Feature Pack gets its name! The purpose of a Feature is to be a configurable, composable block of markup on your webpage that can display content fetched from your content sources (although it doesn't have to). Features get defined in code in your Feature Pack, then added to a page or template by editors in the PageBuilder Admin.

Since we don't know which Features might exist on a page or template when we define them, it's a good idea to make your Feature code entirely self-sufficient - that is, as much as possible it shouldn't rely on the presence or absence of any other components on the page (this includes Layouts, Chains, and other Features).

Writing a feature
-----------------

Let's define a simple Feature whose purpose is to display the title and details for a movie we've fetched from our content source. We'll create a new directory named `movies` in the `/components/features/` directory, and a file within that folder called `movie-detail.jsx`.

> **NOTE**
> 
> Feature components, unlike other components, are stored in directories based on their "group". A "group" is just a namespace of related Features - you might have an `article` group that contains `headline` and `mainStory` Features, for example. Here, the "group" we're creating is called `movies`, so we create a directory with that name and store our `MovieDetail` Feature inside of it.

The Feature might look like this:

    /*  /components/features/movies/movie-detail.jsx  */
    
    import React, { Component } from 'react'
    
    class MovieDetail extends Component {
      render () {
        return (
          <div className='movie-detail col-sm-12 col-md-8'>
            <h1>Jurassic Park</h1>
            <p><strong>Director:</strong> Steven Spielberg</p>
            <p><strong>Actors:</strong> Sam Neill, Laura Dern, Jeff Goldblum, Richard Attenborough</p>
            <p><strong>Plot:</strong> Lorem ipsum</p>
            <p><strong>Rated:</strong> PG-13</p>
            <p><strong>Writer:</strong> Michael Crichton (novel), Michael Crichton (screenplay), David Koepp (screenplay)</p>
            <p><strong>Year:</strong> 1993</p>
            <img src='https://m.media-amazon.com/images/M/MV5BMjM2MDgxMDg0Nl5BMl5BanBnXkFtZTgwNTM2OTM5NDE@._V1_SX300.jpg' alt={`Poster for Jurassic Park`} />
          </div>
        )
      }
    }
    
    MovieDetail.label = 'Movie Detail'
    
    export default MovieDetail
    

After creating this file, you should be able to go to the Page Editor in PageBuilder Admin and add this Feature to the `homepage` we created earlier - click on `Add to` button under the Layout section to add your new feature. Once you do, you should see it displayed on the page! Notice that the value set for `MovieDetail.label` is the name of the Feature displayed in the Page Editor. The `label` allows you to display this Feature in the PageBuilder Editor with a more human-friendly name. You could also set the value of `label` to be an Object that maps i18n locale codes to their translations:

    MovieDetail.label = {
      en: 'Movie Detail',
      es: 'Los detalles de la pelicula'
    }
    

If you provide a label in this format, the Feature will automatically be translated to either English or Spanish. If a translation is not provided for a locale, the Feature will be displayed using the filename. In this case, a web browser set to French will fall back to displaying `MovieDetail`.

> **NOTE**
> 
> Remember, if you don't see your component show up right away, you may need to restart your Fusion app with `CTRL+C` and then re-running `npx fusion start`!

While this very simple component will work, it doesn't really solve our problem since it doesn't render any dynamic content; just static information about a [single (awesome) movie](https://www.imdb.com/title/tt0107290/). 

Handling different output types
-------------------------------

Often times, you'll want your components to operate differently based on what Output Type they are being rendered within. This is useful, for example, in situations where you want to render a desktop web version of a Feature, and then a different mobile web version, and maybe even a version for Google AMP consumption. There are two main ways to achieve this result:

#### Different component versions per output types

It's possible to define entirely different versions of a Feature depending on what output type is being rendered. You can utilize this functionality by naming your feature components after the output types they should correspond to, and then putting them all inside that Feature's specific directory.

In the example above, we created a `movies/` directory and added a `movie-detail.jsx` file to it, indicating that this version of our Feature should be used with every outputType. If we wanted to create a different version of the feature for _each_ outputType we had, we could make `movie-detail` a directory instead of a file. Then, within that directory we could create a `default.jsx` file that served as our "default" version of the component. If we then wanted to serve a different version of the component for Amp, and we had an output type named `amp`, we'd simply add an `amp.jsx` file to our `movie-detail/` directory and write whatever AMP-specific Feature code we wanted in it.


#### Same output type with different logic

Alternatively, if a Feature should act very similarly across multiple Output Types, but has only a few small changes between each, you can simply render a single Feature component for all Output Types and add logical changes based on the `props.outputType` property proved by the Consumer. For example:

      render() {
        const { globalContent, outputType } = this.props
    
        if (outputType === 'amp') {
          return (
            <a href="/link-to-webpage-that-does-something">Click Me!</a>
          )
        }
    
        return <a onClick={ this.doSomething }>Click Me!</a>
      }
    

In this example, we render slightly different links if the provided `outputType` is `amp` or not. Since AMP doesn't allow JavaScript, we send the user to a link - in all other cases, some client side JS is invoked.

It's up to you whether it makes sense to create an entirely new version of a Feature for a different output type, or if the changes are small enough that they can be contained in 1 Feature definition.