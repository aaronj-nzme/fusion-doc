Feature API
===========

Implementation
--------------

##### Naming

A Feature is expected to be stored and named in one of the following formats:

*   `/components/features/{featureGroup}/{featureName}.(js|jsx)`

> This will build one version of this component that is rendered for all Output Types, where the `{featureCategory}` portion of the filepath represents a namespace of related Features, and `{featureName}` represents the name of this Feature.

*   `/components/features/{featureGroup}/{featureName}/{outputTypeName}.(js|jsx)`

> This will build a version of this component that corresponds to the name of the Output Type in the `{outputTypeName}` portion of the filename. The `{featureCategory}` portion of the filepath represents a namespace of related Features, and `{featureName}` represents the name of this Feature. If there is a component named `default.(js|jsx)`, that component will be rendered as a fallback if no file with the name of the relevant Output Type is found.

##### Example

    /*  /components/features/article/headline.jsx  */
    
    import Consumer from 'fusion:consumer'
    import PropTypes from 'prop-types';
    import React from 'react'
    
    @Consumer
    const Headline = (props) => {
      const { headline, byline } props.globalContent
      const { showByline }
      return (
        <h1>{headline}</h1>
        {showByline &&
          <h3>{byline}</h3>
        }
      )
    }
    
    Headline.propTypes = {
      customFields: PropTypes.shape({
        showByline: PropTypes.bool.isRequired
      })
    }
    
    export default Headline
    

Props
-----

### displayProperties (Object)

##### Description

`displayProperties` is an object whose names and types are defined per-Output-Type by the [<code>displayPropTypes</code>](api-output.md#displayPropTypes) , and whose values are then set in PageBuilder. The `displayProperties` object is intended to be used for display-related properties such as column sizes, hide/show logic and more that may be specific to the Output Type this component is rendering in.

##### Example

    /*  /components/features/global/footer/amp.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      const { fullWidth } = props.displayProperties
    
      return (
        <footer className={fullWidth ? 'col-sm-12' : null}>
          <p>&copy; 2018 Acme Corp.</p>
        </footer>
      )
    }
    

Static Values
-------------

### label (Object or String)

##### Description

The `label` field is used in the PageBuilder Editor instead of the actual component filename. The primary purpose of this value is to enable internationalization (i18n) for your Feature. If this Feature component will be used by publications in multiple languages, a `label` allows the PageBuilder Editor to use the translation provided for each locale.

##### Example

Providing a `label` as an Object is the preferred approach as it enables internationalization for any locales that are provided in the component implementation.

    /*  /components/features/article/headline.jsx  */
    
    import Consumer from 'fusion:consumer'
    import PropTypes from 'prop-types';
    import React from 'react'
    
    @Consumer
    const Headline = (props) => {
      const { headline, byline } props.globalContent
      const { showByline }
      return (
        <h1>{headline}</h1>
        {showByline &&
          <h3>{byline}</h3>
        }
      )
    }
    
    Headline.label = {
      en: "Headline",
      fr: "Le titre"
    }
    
    Headline.propTypes = {
      customFields: PropTypes.shape({
        showByline: PropTypes.bool.tag({ label: {
          en: "Show byline",
          fr: "Montrer à l'auteur"
        }}).isRequired
      })
    }
    
    export default Headline
    

Providing a `label` as a String is also supported for Features that should always have the same label in the PageBuilder Editor. If a `label` is provided as a String, then the value will always be used even if the user is in a different locale.

    /*  /components/features/article/headline.jsx  */
    
    import Consumer from 'fusion:consumer'
    import PropTypes from 'prop-types';
    import React from 'react'
    
    @Consumer
    const Headline = (props) => {
      const { headline, byline } props.globalContent
      const { showByline }
      return (
        <h1>{headline}</h1>
        {showByline &&
          <h3>{byline}</h3>
        }
      )
    }
    
    Headline.label = `Le titre de l'article`
    
    Headline.propTypes = {
      customFields: PropTypes.shape({
        showByline: PropTypes.bool.tag({ label: `Montrer à l'auteur` }).isRequired
      })
    }
    
    export default Headline
    

Custom Fields
-------------

See the [Custom Fields documentation](./api-custom-fields.md)