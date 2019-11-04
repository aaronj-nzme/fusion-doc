
Working with Display Properties
===============================

One of the things that makes PageBuilder and the Fusion Engine unique is how it can be adapted to serve multiple different Output Types. Output Types allow us to display content in a variety of different formats without refactoring all of the components included within them (for clarity, in this document "components" will mean Features, Chains and Layouts). Common use cases for different Output Types include a standard desktop Output Type, a separate "mobile web" Output Type, and different Output Types for proprietary platforms such as [Google Amp](https://www.ampproject.org/), [Facebook Instant Articles](https://instantarticles.fb.com/) and more.

While Output Types are useful, they do present a challenge: how can we write components that are generic enough to work in _every_ Output Type, but also able to be rendered differently in _each_ Output Type depending on our needs? The answer, as you may have guessed, is "Display Properties".

What are display properties?
----------------------------

Display Properties are a set of keys defined in the Output Type whose values are set on a per-component basis in the PageBuilder Admin. Another way to think of Display Properties is "custom fields, but defined per Output Type". Using this pattern, we can write our components generically and then define any qualities that are specific to an Output Type with a Display Property.

Defining in the Output Type
---------------------------

For example, let's say in addition to our `default` Output Type, we also want to create a `mobile` Output Type that (among other things) selectively hides or shows which content should fit in a mobile view. We can create that new Output Type by adding a `mobile.jsx` file to the `/components/output-types/` directory. Since the JSX definition of our `mobile` Output Type doesn't actually differ from that of our `default` Output Type, we can simply `import` the default Output Type into our `mobile` one for now, and extend it later.

    /*  /components/output-types/mobile.jsx  */
    
    import OutputType from './default.jsx'
    

However, we also want to add some `displayPropTypes` to our Output Type before actually exporting it. `displayPropTypes` are the way we define the "keys" of our Display Properties, which become form items that can be set on a per-component basis in PageBuilder. We can add our `displayPropTypes` by importing the `PropTypes` library and defining the property that we care about inside a static property called `displayPropTypes`, which is an object:

    /*  /components/output-types/mobile.jsx  */
    import PropTypes from 'prop-types'
    import OutputType from './default.jsx'
    
    OutputType.displayPropTypes = {
      hideOnMobile: PropTypes.bool.tag({
        name: 'Hide on Mobile?'
      })
    }
    
    export default OutputType
    

Above, we've defined a boolean property named `hideOnMobile` which we can use to determine whether each of our components should be hidden in a mobile Output Type. Since `displayPropTypes` use the same Fusion-specific version of the PropTypes library as Custom Fields do, we can also chain the \<code\>.tag\<\/code\> method to our property. We pass an object containing a `name` property as the argument, which is the human-readable version of the key; this will be the label that editors see in PageBuilder.

Using display properties in a component
---------------------------------------

Now that we've defined the property we care about in our Output Type, we can see the option in our PageBuilder instance. When we open the editor view of our "homepage" and select the `MovieList` component in the sidebar, and then select the "Display" tab in the sidebar, we will see our "Hide on Mobile?" checkbox is displayed! Why don't we set this to `true` for now by selecting the checkbox on the `MovieList` component, so it won't be shown in the mobile view.

Even though we can now set this value for each of our components, we aren't actually using it yet. That's because we need to write some code in each of our components that consumes the `displayProperties` prop, which will contain the value set in PageBuilder.

That's easy enough to do - we just have to use the `props.displayProperties` object in a `Consumer` wrapped component, and we'll have access to the value we need. Let's see how we can use that in our `MovieList` component:

    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class MovieList extends Component {
    
      ...
    
      render () {
        // Extract the `hideOnMobile` value from props.displayProperties, which we default to an empty object
        const { hideOnMobile } = this.props.displayProperties || {}
        // Before anything else, if hideOnMobile is true, we return null so nothing else gets rendered
        if (hideOnMobile) return null
    
        ...
      }
    }
    
    export default MovieList
    

Since `hideOnMobile` is a boolean, we simply need to extract it from the `displayProperties` object and check for its existence; if it's `true`, we want to return `null` from this component so nothing gets rendered.

Despite the simplicity of this example, hopefully you can see how Display Properties can be used to provide differentiated logic between multiple Output Types in a single component.