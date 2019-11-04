
Adding Custom Fields to Components
==================================

Custom Fields are an especially useful tool when building websites with Fusion and PageBuilder. As their name suggests, Custom Fields allow developers and editors to set arbitrary (i.e. "custom") data (i.e. "fields") on individual instances of Features and Chains in Fusion, which can then be used to change the look, feel or behavior of that particular component.

What are Custom Fields?
-----------------------

Custom Fields are simply key/value pairs assigned to an individual Feature or Chain. The key names are defined in their respective components, and the values for each component instance are set in the PageBuilder Admin. This allows Feature Pack developers to define and use data relevant to their components, and for PageBuilder editors to decide how individual components should behave.

Common use cases for Custom Fields include setting headings or text that should be customizable for each Feature, styling attributes, and content configurations that allow PageBuilder editors to assign content sources to components.

Custom Fields as PropTypes
--------------------------

Custom Fields can be added to either Features or Chains in a Feature Pack. Both define Custom Fields in the same way - using React's [PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html) standard to denote the name and type of data the Custom Field is expecting.

Custom Fields can be added to both functional components and class-based components. Here's an example using the `MovieDetail` component we defined in the "Creating a Feature Component" guide:

    /*  /components/features/movies/movie-detail.jsx  */
    
    import PropTypes from 'prop-types'
    import Consumer from 'fusion:consumer'
    import React, { Component, Fragment } from 'react'
    
    @Consumer
    class MovieDetail extends Component {
      render () {
        const { Actors, Director, Plot, Poster, Rated, Title, Writer, Year } = this.props.globalContent
    
        // We can extract our custom field values here, and even set default values if desired...
        const { moviePrefix = 'Movie', showExtendedInfo = false } = this.props.customFields
    
        return (
          <div className='movie-detail col-sm-12 col-md-8'>
            {/* We use the `moviePrefix` value before the Title */}
            <h1><span>{moviePrefix}:</span> {Title}</h1>
            ...
            {/* we can use our boolean value `showExtendedInfo` to determine if certain data gets displayed or not */}
            {showExtendedInfo &&
              <Fragment>
                {Rated && <p><strong>Rated:</strong> {Rated}</p>}
                {Writer && <p><strong>Writer:</strong> {Writer}</p>}
                {Year && <p><strong>Year:</strong> {Year}</p>}
              </Fragment>
            }
            ...
          </div>
        )
      }
    }
    
    MovieDetail.propTypes = {
      customFields: PropTypes.shape({
        moviePrefix: PropTypes.string.isRequired,
        showExtendedInfo: PropTypes.bool
      })
    }
    
    export default MovieDetail
    

As you can see in the code and comments above, we defined a required `moviePrefix` custom field that should contain some text to prefix our movie title, and an optional `showExtendedInfo` field that is a boolean determining whether to show certain data in this view. These values will now be configurable in the PageBuilder Admin by editors, and we can use them just like any other data in our component to change its behavior!

Inline Editing
--------------

When we're adding custom fields to a component, it's easy for us as developers to think of how that custom field will work and what changing it will do to the component. But remember: PageBuilder is built for editors too! Our editors might not know what our `moviePrefix` custom field is meant to do, or any other custom field for that matter.

To make it clearer and easier for editors to configure custom fields, PageBuilder has "editable" custom fields. An editable field is one that can be edited inline from our "preview" in PageBuilder on the rendered page itself, rather than from a panel in the sidebar.

However, custom fields aren't inline editable by default; it's up to the Feature Pack developer to denote a custom field that should be inline editable, and which DOM element it is tied to. We can do that using the `editableField` prop that gets passed to `Consumer` decorated components. It's a function takes the name of the field we want to edit as a string, and returns an array of attributes that PageBuilder uses to identify the element. Rather than printing the attributes out manually, it is considered best practice to use the [ES6 spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) to enumerate the attributes directly on the element we want to edit.

    /*  /components/features/movies/movie-detail.jsx  */
    
    ...
    @Consumer
    class MovieDetail extends Component {
      render () {
        return (
          <div className='movie-detail col-sm-12 col-md-8'>
            {/* Invoking the `editableField` method with `moviePrefix` as the custom field it is tied to */}
            <h1><span {...this.props.editableField('moviePrefix')}>{moviePrefix}:</span> {Title}</h1>
            ...
          </div>
        )
      }
    }
    
    MovieDetail.propTypes = {
      customFields: PropTypes.shape({
        moviePrefix: PropTypes.string,
        showExtendedInfo: PropTypes.bool
      })
    }
    
    export default MovieDetail
    

In the above code, the `<span>` that contains our `moviePrefix` content will now be editable in the Admin Preview. When a PageBuilder editor focuses on the field and edits, whatever they type inside the `<span>` will be saved as the new value of the `moviePrefix` custom field.
