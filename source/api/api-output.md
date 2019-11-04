Output Type API
===============

Output Types are used to define the HTML wrapping that is applied to server responses (think `head`, `title`, `meta`, `link`, `script` tags, etc). Anything that is part of the server generated HTML, but is not specific to a page/template should be defined in an output type.

When rendering a page, the `outputType` query parameter will be used to determine which output type is used to handle the request; if no parameter is specified, `default` will be used.

Implementation
--------------

##### Naming

An Output Type is expected to be stored and named in the following format:

*   `/components/output-types/{outputTypeName}.(js|jsx)`

> This will build an Output Type component where the `{outputTypeName}` portion of the filepath represents the name of the Output Type.
> 
> **NOTE**
> 
> Fusion requires that your Output Type contain an element in the body that has an ID value of `fusion-app`, which contains the value `props.children` inside of it. This is the ID that Fusion will render the client side portion of the application into. Without this ID, client-side rendering will fail.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            <title>{props.metaValue('title') || 'Default Title'}</title>
            <props.MetaTags />
            <props.Libs />
            <props.CssLinks />
            <link rel='icon' type='image/x-icon' href={`${props.contextPath}/resources/img/favicon.ico`} />
          </head>
          <body>
            <h1>Welcome to Fusion!</h1>
            <div id='fusion-app'>
              {props.children}
            </div>
            <props.Fusion />
          </body>
        </html>
      )
    }
    

Props
-----

All props that are made available by the `Context` component are also available on Output Types. Consult the [Context API docs](./api-context.md#props) for a list of these props.

### children - (Array)

##### Description

React uses the convention of a prop named `children` to define content that should be inserted inside your component. We respect this convention for output types. In this case, the `children` prop defines the rendered content of the page. If you want to perform client-side-only rendering, do not include this property in your page.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        ...
        <div id='fusion-app'>
          {props.children}
        </div>
        ...
      )
    }
    

### metaValue() - (Function)

##### Description

Similar to `MetaTag` component below, but returns only the value, not a fully rendered HTML meta tag.

##### Parameters

`metaValue(name, [defaultValue])`

##### Return

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        ...
        <title>{props.metaValue('title') || 'Default Title'}</title>
        ...
      )
    }
    

Prop Components
---------------

### props.CssLinks - (Component)

Also aliased as `<props.CssTags />`

##### Description

Includes the stylesheet `<link>`s for the appropriate page/template and/or output-type. Will insert a `<link>` reference tag. In the case of page/template, the file name will be content hashed to the specific state of the page or template. For inline styles, see [<code><props.Styles /></code>](#styles) below.

##### Parameters

The `CssLinks` component can be invoked on its own, or with [render props](https://reactjs.org/docs/render-props.html). If invoked on its own, it will render `<link>` tags for both the Output Type stylesheet as well as the template-specific stylesheet.

`<props.CssLinks />`

If invoked with a render prop, it will accept a function that gets passed the URLs for both the Output Type stylesheet and template-specific stylesheet passed to it, which the function can then use to render the `<link>` tags on its own.

`<props.CssLinks>{renderProp}</props.CssLinks>`

*   `renderProp(stylesheetObj)` (_Function_): Function whose purpose is to render `<link>` tags, given an object containing stylesheet URLs.
    *   `stylesheetObj` (_Object_): A wrapper object containing the `outputTypeHref` and `templateHref` key/value pairs.
    *   `stylesheetObj.outputTypeHref` (_String_): The stylesheet URL for CSS included in the Output Type itself.
    *   `stylesheetObj.templateHref` (_String_): The stylesheet URL for CSS included in the components specific to this page/template implementation.

##### Return

Renders the return value of the passed `renderProp` if one is provided; otherwise, renders `<link>` tags for the Output Type and specified template.

##### Example

_No Render Props_

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        ...
        <props.CssLinks />
        ...
      )
    }
    

_With Render Props_

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        ...
        <props.CssLinks>
          {({outputTypeHref, templateHref}) => {
            // Only rendering the outputTypeHref, not the templateHref
            return (<link rel='stylesheet' type='text/css' href={outputTypeHref} />)
          }}
        </props.CssLinks>
        ...
      )
    }
    

> **NOTE**
> 
> Customizing reference links is not recommended unless absolutely necessary. If inserting a custom link to the page/template styles, please be sure to include `id="fusion-template-styles"` since that ID is used to update the styles in the case of a newly-published template.

### props.Fusion - (Component)

##### Description

Include a special script block that includes Fusion-specific values, as well as the server-fetched content cache that is used during client-side refresh to prevent content flashing.

Content refresh may be specifically disabled. This only applies to content that was previously fetched server-side and is available in the content cache; any content that is not in the cache will be fetched from the client, regardless. This feature is generally only useful in the case of prerendered HTML. On-demand HTML is already aware that its content is fresh and will not attempt client-side fetching.

##### Parameters

None

##### Return

Renders bootstrapped Fusion-specific values, as well as cached content fetched from the server side, to the DOM for Fusion's use.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
          </head>
          <body>
            ...
            <props.Fusion />
          </body>
        </html>
      )
    }
    

### props.Libs - (Component)

##### Description

Include the scripts necessary to perform client-side re-render and content refresh. This includes both a shared react script and a specific page/template script.

##### Parameters

None

##### Return

Renders the React engine script and Fusion template script necessary to re-hydrate and re-render the React app client side.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
            <props.Libs />
            ...
          </head>
          ...
        </html>
      )
    }
    

### props.MetaTag - (Component)

Also aliased as `<props.MetaTags />`

##### Description

This component will insert meta tags into your rendered HTML. If name is provided, it will insert the single meta tag specified. If no name is given, it will insert all meta tags that are enabled in the admin.

##### Parameters

Parameters are passed to the MetaTag component as props.

`<props.MetaTag [name] [default] />`

*   `name` (_String_): A string representing the name of the `<meta>` tag to be rendered; this should correspond to a meta option set in the PageBuilder admin.
*   `default` (_String_): The default value this `<meta>` tag should contain if a value has not been set in PageBuilder.

##### Return

Renders the specified `<meta>` tag by name if one is provided; if not, renders all `<meta>` tags for this page.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
            <props.MetaTag name='description' default='This is a page about cats.' />
            ...
          </head>
          ...
        </html>
      )
    }
    

### props.Resource - (Component)

##### Description

This component provides access to the content of static resource files.

##### Parameters

Parameters are passed to the Resource component as props.

`<props.Resource [path] [encoding] />`

*   `path` (_String_): A string representing the path of the file to be accessed.
*   `encoding` (_String_): The preferred encoding of the resultant string (should be one of `ascii`, `base64`, `binary`, `buffer`, `hex`, or `utf8`; default is `utf8`).

##### Return

Renders the return value of the passed `renderProp`.

##### Example

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
            <props.Resource path='resources/css/main.css'>
              {({ data }) => data ? <style>{data}</style> : null}
            </props.Resource>
            ...
          </head>
          ...
        </html>
      )
    }
    

### props.Styles - (Component)

##### Description

Include the generated CSS for the appropriate page/template and/or output-type. Will insert the computed inlined styles appropriate to the rendering. For a reference link, see [<code><props.CssLinks /></code>](#props.CssLinks) below.

##### Parameters

The `Styles` component can be invoked on its own, or with [render props](https://reactjs.org/docs/render-props.html). If invoked on its own, it will render `<style>` tags including the CSS for both the Output Type stylesheet as well as the template-specific stylesheet.

`<props.Styles />`

If invoked with a render prop, it will accept a function that gets passed the URLs for both the Output Type stylesheet and template-specific stylesheet passed to it, which the function can then use to render the `<link>` tags on its own.

`<props.Styles>{renderProp}</props.Styles>`

*   `renderProp(stylesObj)` (_Function_): Function whose purpose is to render `<link>` tags, given an object containing stylesheet URLs.
    *   `stylesObj` (_Object_): A wrapper object containing the `outputTypeStyles` and `templateStyles` key/value pairs.
    *   `stylesObj.outputTypeStyles` (_String_): The CSS for the Output Type itself.
    *   `stylesObj.templateStyles` (_String_): The CSS for the components specific to this page/template implementation.

##### Return

Renders the return value of the passed `renderProp` if one is provided; otherwise, renders `<style>` tags for the Output Type and specified template containing relevant CSS.

##### Example

_No Render Prop_

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
            <props.Styles />
            ...
          </head>
          ...
        </html>
      )
    }
    

_With Render Prop_

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    export default (props) => {
      return (
        <html>
          <head>
            ...
            <props.Styles>
              {({outputTypeStyles, templateStyles}) => {
                // Only rendering the outputTypeStyles, not the templateStyles
                return (<style amp-custom='true'>{outputTypeStyles}</style>)
              }}
            </props.Styles>
            ...
          </head>
          ...
        </html>
      )
    }
    

Static Values
-------------

### contentType - (String)

##### Description

Provide a specific value to be used as the `Content-Type` header in the response. If no value is provided, will default to `text/html` if render results in a string, or `application/json` otherwise.

##### Example

    /*  /components/output-types/json.js  */
    
    const JsonOutputType = (props) => {
      ...
    }
    
    JsonOutputType.contentType = 'application/json'
    
    export default JsonOutputType
    

### displayPropTypes - (Object)

##### Description

Implements React's [PropTypes](https://github.com/facebook/prop-types) which will be available on each feature. These are defined the same way as `propTypes` on Features and Chains however, unlike custom fields, the values can differ across Output Types. `displayProperties` are useful for high level page functionality such as hide/show logic, layout grids, or any other settings that you want each feature to have.

The value [<code>props.displayProperties</code>](./api-feature.md#displayProperties) will be available on Features, and contain the key names defined here and the values set in PageBuilder.

##### Example

On the Output Type

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    const DefaultOutputType = (props) => {
      ...
    }
    
    DefaultOutputType.displayPropTypes = {
      isStatic: PropTypes.bool,
      width: PropTypes.number,
      anchor: PropTypes.string
    }
    
    export default DefaultOutputType
    

### fallback - (Boolean or Array of Strings)

##### Description

When rendering, Fusion must determine which version of a component should be rendered relative to the Output Type being constructed (in this case _component_ means Features, Chains or Layouts). This property determines how (if at all) child components within a certain Output Type get rendered.

By default, Fusion will select the version of the component that is named after the relevant Output Type and its file extension (e.g. if the Output Type being rendered is `amp.jsx`, it looks for a component file named `amp.jsx`). If only one version of a component exists (e.g. a single `default.jsx` file), by default all other Output Types will fallback to that single version of the component. Using the `fallback` property, however, a developer can specify:

*   that child components of this Output Type should not render at all if there is no corresponding version of the component, or
*   an array of "fallback" versions for the given Output Type that Fusion should attempt to render, in order, if there is no corresponding version of the component

The options above use the _Boolean_ version or the _Array of Strings_ version of this property, respectively.

##### Example

_Boolean Version_

    /*  /components/output-types/amp.jsx  */
    
    import React from 'react'
    
    const AmpOutputType = (props) => {
      ...
    }
    
    // If no `amp` version exists for a given child component of this Output Type, that component will not render
    AmpOutputType.fallback = false
    
    export default AmpOutputType
    

_Array of Strings Version_

    /*  /components/output-types/fb-instant.jsx  */
    
    import React from 'react'
    
    const FbInstantOutputType = (props) => {
      ...
    }
    
    // If no `fb-instant` version exists for a given child component of this Output Type, that component will fallback to the `amp` version if it exists, then the `default` version
    FbInstantOutputType.fallback = ['amp', 'default']
    
    export default FbInstantOutputType
    

### transform - (Object)

##### Description

Extend an output-type to provide separate output-types that are post-render modifications of the original. The transform object should consist of functions whose names are unique output-type names. When a request is made where the output-type is specified as one of these transform types, a render will be performed using the base OutputType component. The transform function will then be called, with the render data and context passed as arguments. The transform function should return an object with a `data` property that will be used as the body of the response, and optionally a `contentType` property to be used as the `Content-Type` header.

##### Example

The following example results in two available output-types: `default` and `arcio`. Each will perform a render using the DefaultOutputType component. A request for `?outputType=default` will return the rendered HTML, while a request for `?outputType=arcio` will return a JSON object with context information.

    /*  /components/output-types/default.jsx  */
    
    import React from 'react'
    
    const DefaultOutputType = (props) => {
      ...
    }
    
    DefaultOutputType.contentType = 'text/html'
    
    DefaultOutputType.transform = {
      arcio ({ context, data}) {
        return {
          contentType: 'application/json',
          data: {
            tree: context.tree,
            globalContent: context.globalContent,
            featureContent: context.contentCache
          }
        }
      }
    }
    
    export default DefaultOutputType