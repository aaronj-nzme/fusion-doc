Chain API
=========

Chains are Fusion components that serve as wrapping elements around a group of Features. Features are grouped into a Chain by editors in PageBuilder, and are available to the component as [<code>props.children</code>](#children). Chains are rendered both on the server and the client (i.e. isomorphically). Chains also support [custom fields](#custom-fields), and can be rendered differently per Output Type.

Implementation
--------------

##### Naming

A Chain is expected to be stored and named in one of the following formats:

*   `/components/chains/{chainName}.(js|jsx)`

> This will build one version of this component that is rendered for all Output Types, where the `{chainName}` portion of the filepath represents the name of the Chain.

*   `/components/chains/{chainName}/{outputTypeName}.(js|jsx)`

> This will build a version of this component that corresponds to the name of the Output Type in the filename. The `{chainName}` portion of the filepath represents the name of the Chain. If there is a component named `default.(js|jsx)`, that component will be rendered as a fallback if no file with the name of the relevant Output Type is found.

##### Example

    /*  /components/chains/two-chainz.jsx  */
    
    import React from 'react'
    
    const TwoChainz = (props) => {
      const mid = Math.floor(props.children.length / 2)
      const firstCol = props.children.slice(0, mid)
      const secondCol = props.children.slice(mid)
    
      return (
        <div>
          <section>{firstCol}</section>
          {secondCol && secondCol.length > 0 &&
            <section>{secondCol}</section>
          }
        </div>
      )
    }
    
    export default TwoChainz
    

Props
-----

### children - (Array)

See the `children` section in the [Output Type API](./api-output.md#children)

### childProps - (Array)

See the `childProps` section in the [Layout API](./api-layout.md#childProps)

Static Values
-------------

### label (Object or String)

##### Description

The `label` field is used in the PageBuilder Editor instead of the actual chain filename. The primary purpose of this value is to enable internationalization (i18n) for chains. If this chain will be used by publications in multiple languages, a `label` allows the PageBuilder Editor to use the translation provided for each locale.

##### Example

Providing a `label` as an Object is the preferred approach as it enables internationalization for any locales that are provided in the chain implementation. In this example, users in an English-speaking locale will see `Two chains`, but users in a Spanish-speaking locale will see `Dos cadenas`.

    /*  /components/chains/two-chainz.jsx  */
    
    import React from 'react'
    
    const TwoChainz = (props) => {
      const mid = Math.floor(props.children.length / 2)
      const firstCol = props.children.slice(0, mid)
      const secondCol = props.children.slice(mid)
    
      return (
        <div>
          <section>{firstCol}</section>
          {secondCol && secondCol.length > 0 &&
            <section>{secondCol}</section>
          }
        </div>
      )
    }
    
    TwoChainz.label = {
      en: 'Two chains',
      es: 'Dos cadenas'
    }
    
    export default TwoChainz
    

Providing a `label` as a String is also supported for chains that should always have the same label in the PageBuilder Editor. If a `label` is provided as a String, then the value will always be used even if the user is in a different locale. In this example, users will always see `2 Chains` regardless of their locale.

    /*  /components/chains/two-chainz.jsx  */
    
    import React from 'react'
    
    const TwoChainz = (props) => {
      const mid = Math.floor(props.children.length / 2)
      const firstCol = props.children.slice(0, mid)
      const secondCol = props.children.slice(mid)
    
      return (
        <div>
          <section>{firstCol}</section>
          {secondCol && secondCol.length > 0 &&
            <section>{secondCol}</section>
          }
        </div>
      )
    }
    
    TwoChainz.label = `2 Chains`
    
    export default TwoChainz
    

Custom Fields
-------------

See the [Custom Fields documentation](./api-custom-fields.md)