Context Component API
=====================

Fusion `<Context>` is a React component for accessing the static context properties.

There are 2 fusion contexts: the application context is globally scoped and includes properties that apply to the entire rendered page; the component context includes properties that apply to the closest ancestor component.

Implementation
--------------

The Fusion `<Context>` component takes no arguments and uses the render props pattern to call either a provided render property, or child function(s), with the static properties that include both the app- and component-scoped context properties.

### Context Component

The `<Context>` component is imported from the `fusion:context` namespace. It is the default export of the module and should be used as a React Component.

##### Example

    import React from 'react'
    import Context from 'fusion:context'
    
    const MyFeatureComponent = (props) =>
      <Context>
        {({ arcSite, id }) =>
          <>
            <div>arc-site: {arcSite}</div>
            <div>feature id: {id}</div>
          </>
        }
      </Context>
    
    export default MyFeatureComponent
    

The children array is expected to contain valid React components, so you may refactor to use more convenient child components.

    import React from 'react'
    import Context from 'fusion:context'
    
    const ArcSiteComponent = ({ arcSite }) => <div>arc-site: {arcSite}</div>
    const ComponentIdComponent = ({ id }) => <div>feature id: {id}</div>
    
    const MyFeatureComponent = (props) =>
      <Context>
        {ArcSiteComponent}
        {ComponentIdComponent}
      </Context>
    
    export default MyFeatureComponent
    

If using a functional component, you may also access the same information with React hooks as follows:

    import React from 'react'
    import { useFusionContext } from 'fusion:context'
    
    function ArcSiteComponent () {
      const { arcSite, id } = useFusionContext()
    
      return <>
        <div>arc-site: {arcSite}</div>
        <div>feature id: {id}</div>
      </>
    }
    
    export default ArcSiteComponent
    

### AppContext and ComponentContext Components

There are app- or compontent-scoped context components available as `AppContext` or `ComponentContext`, respectively.

##### Example

    import React from 'react'
    import { AppContext, ComponentContext } from 'fusion:context'
    
    const ArcSiteComponent = ({ arcSite }) => <div>arc-site: {arcSite}</div>
    const ComponentIdComponent = ({ id }) => <div>feature id: {id}</div>
    
    const MyFeatureComponent = (props) =>
      <>
        <AppContext>
          {ArcSiteComponent}
        </AppContext>
        <ComponentContext render={ComponentIdComponent} />
      </>
    
    export default MyFeatureComponent
    

or you may access app- or component-scoped context using hooks as follows:

    import React from 'react'
    import { useAppContext, useComponentContext } from 'fusion:context'
    
    function ArcSiteComponent () {
      const { arcSite } = useAppContext()
      const { id } = useComponentContext()
    
      return <>
        <div>arc-site: {arcSite}</div>
        <div>feature id: {id}</div>
      </>
    }
    
    export default ArcSiteComponent
    

### HOCs

The `<Context>`, `<AppContext>`, and `<ComponentContext>` components may also be used as higher-order components to have the corresponding context properties appended to the incoming props of your custom component. These HOCs are also available as with-prefixed, if you prefer the redux naming style (e.g., `withAppContext`, `withComponentContenxt`, or `withFusionContext`)

##### Example

    import React from 'react'
    import FusionContext from 'fusion:context'
    
    const MyFeatureComponent = (props) =>
      <>
        <div>arc-site: {props.arcSite}</div>
        <div>feature id: {props.id}</div>
      </>
    
    export default FusionContext(MyFeatureComponent)
    

Props
-----

### App Context

#### arcSite (String)

##### Description

The Arc site used in this rendering, if multi-site enabled. This will be determined by the value of the `_website` query parameter appended to the page request.

##### Example

    /*  /components/features/global/footer.jsx  */
    
    import React from 'react'
    import Context from 'fusion:context'
    
    const Footer = ({ arcSite }) => <p>&copy; 2018 {arcSite}</p>
    
    export default (props) =>
      <Context>
        {Footer}
      </Context>
    

#### contextPath (String)

##### Description

This is the base context path of the application. In the client, you could calculate this using `window.location`, but this property exists to provide similar server-side access.

##### Example

    /*  /components/features/global/logo.jsx  */
    import React from 'react'
    import Context from 'fusion:context'
    
    const Logo = ({ contextPath }) => <img src={`${contextPath}/resources/img/logo.png`} />
    
    export default (props) =>
      <Context>
        {Logo}
      </Context>
    

#### deployment() - (Function)

Also aliased as `version()`

##### Description

Appends a given resource path with the deployment ID (of the lambda function) that is generating this response. Should be called as a function receiving the path you want to have modified as `deployment('/pf/resources/styles/main.css')`, but can also be added as a query param to static paths as `?d=${deployment}`. Any URLs generated by fusion (e.g., via `CssLinks`) will already be annotated with this param.

> **NOTE**
> 
> The `deployment()` function should be used to wrap paths to request _any_ static resources contained in your bundle. The only exception to this is paths referenced from within your CSS files (e.g. `background-image: url(/pf/resources/bg.png);`) \- Fusion will automatically wrap any paths referenced within your CSS/SCSS files for you, since you can't use the `deployment` function there.

##### Parameters

`deployment(resourcePath)`

*   `resourcePath` (_String_): The webroot relative path (including context path) to a static resource in the bundle. For example, an image whose filepath is `/resources/icon.svg` in the bundle would be `deployment('/pf/resources/icon.svg')`. You can use the `contextPath` helper to dynamically replace `pf` with whatever your specific context path is.

##### Return

Returns the input path with a `?d={deploymentID}` query parameter appended to it.

##### Example

    /*  /components/output-types/default.jsx  */
    
    export default ({ contextPath, deployment }) => (
      <html>
        <head>
          ...
          <link rel='icon' type='image/x-icon' href={deployment(`${contextPath}/resources/favicon.ico`)} />
          <link rel='stylesheet' type='text/css' href={deployment(`${contextPath}/resources/styles/main.css`)} />
        </head>
        ...
      </html>
    )
    

#### globalContent (Object)

##### Description

This is the full data object used as the global content for the rendered page.

##### Keys

The keys will be the actual data object returned from the content fetch; as such we don't know what they will be beforehand.

##### Example

    /*  /components/features/article/headline.jsx  */
    
    import React from 'react'
    import Context from 'fusion:context'
    
    const Headline = ({ globalContent }) => {
      const { headline } = globalContent
      return headline && <h1>{headline}</h1>
    }
    
    export default (props) =>
      <Context>
        {Headline}
      </Context>
    

#### globalContentConfig (Object)

##### Description

This is the full config object used to fetch global content for the rendered page.

##### Keys

*   `source` (_String_): This is the name of the content source used to fetch the global content of the page or template.
*   `query` (_Object_): This an object containing the key/value pairs that were used as arguments to fetch content from the global content source.

##### Example

    /*  /components/features/article/story-feed.jsx  */
    
    import React, { Component } from 'react'
    import Consumer from 'fusion:consumer'
    
    const Story = ({ headline }) => <li><h3>{headline}</h3></li>
    
    @Consumer
    class StoryFeed extends Component {
      constructor() {
        super(props)
    
        this.page = 0
        this.state = {
          stories: props.globalContent.stories || []
        }
    
        this.fetchStories = this.fetchStories.bind(this)
      }
    
      fetchStories() {
        this.page += 1
        const { globalContentConfig } = this.props
        // Use the globalContentConfig to fetch stories from the same content source and with the same arguments as the globalContent fetch
        const { fetched } = this.getContent(
          globalContentConfig.story,
          {
            ...globalContentConfig.query,
            page: this.page
          }
        )
    
        fetched.then(({ stories: newStories }) => {
          this.setState(({ stories: existingStories }) => { stories: existingStories.concat(newStories) })
        })
      }
    
      render() {
        const { stories } = this.state
        return (
          <div>
            <ul>
              {stories.map(Story)}
            </ul>
            <button onClick={this.fetchStories()}>More Stories</button>
          </div>
        )
      }
    }
    
    export default StoryFeed
    

#### isAdmin (Boolean)

##### Description

This represents whether the current render is occuring in the PB admin preview, or as a standard page view.

##### Example

    /*  /components/features/global/logo.jsx  */
    
    import React from 'react'
    import Context from 'fusion:context'
    
    const Ad = ({ isAdmin}) => !isAdmin && <div id='some-ad'></div>
    
    export default (props) =>
      <Context>
        {Ad}
      </Context>
    

#### layout (String)

##### Description

The name of the Layout that was used when rendering this page.

##### Example

    /*  /components/features/common/image.jsx  */
    
    import Context from 'fusion:context'
    import React, { Component } from 'react'
    
    const Img = ({ layout }) => {
      // Use the layout prop to determine whether to add a class to our image or not
      const isResponsive = (layout === 'responsive-sidebar')
    
      return <img src={props.imgSrc} className={isResponsive ? 'responsive' : null} />
    }
    
    export default (props) =>
      <Context>
        {Img}
      </Context>
    

#### outputType (String)

##### Description

The Output Type that was used when rendering this page.

##### Example

    /*  /components/features/common/link.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class Link extends Component {
      ...
      render() {
        const { outputType } = this.props
        if (outputType === 'amp') {
          return (<a href={this.props.linkUrl}>{this.props.text}</a>)
        }
    
        return <a onClick={this.invokeJsMethod}>{this.props.text}</a>
      }
    }
    
    export default Link
    

#### renderables - (Array)

##### Description

A flattened array of all component elements from `tree`. Suitable for direct access with `.find`, `.filter`, or `.map`.

#### requestUri (String)

##### Description

This is the URI that was requested to initiate this rendering. In the client, you could access this using `window.location`, but this property exists to provide similar server-side access.

##### Example

    /*  /components/features/common/link.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    import URL from 'url'
    
    @Consumer
    class Link extends Component {
      render() {
        const { requestUri, linkUrl, text } = this.props
    
        // Compare the hostnames of the page that was requested vs. the link to see if the link is to an external domain
        const requestUrlObj = URL.parse(requestUri)
        const linkUrlObj = URL.parse(linkUrl)
        const isDifferentDomain = requestUrlObj.hostname !== linkUrlObj.hostname
    
        // If it's external, open in a new tab
        const targetVal = isDifferentDomain ? '_blank' : null
    
        return <a target={targetVal} href={linkUrl}>{text}</a>
      }
    }
    
    export default Link
    

#### siteProperties (Object)

##### Description

An object containing the site-specific properties defined in the `/properties/` directory for the current [<code>arcSite</code>](#arcsite).

##### Example

    /*  /components/features/header/social-links.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from react
    
    @Consumer
    class SocialLinks extends Component {
      render () {
        const twitter = this.props.siteProperties
    
        return (
          <ul>
            ...
            <li><a href={twitter}>Twitter</a></li>
          </ul>
        )
      }
    }
    
    export default SocialLinks
    

#### template (String)

##### Description

The ID of the template that was used when rendering this page.

#### tree - (Object)

##### Description

A JS object that represents the full component tree of the page to be rendered.

### Component Context

#### collection - (String)

##### Description

The collection of the currently scoped component. Will be one of `layout`, `section`, `chain`, or `feature`.

#### type - (String)

##### Description

The type of the currently scoped component. Will be the name of the component type (e.g., `article/body` or `global/header`).

#### id - (String)

##### Description

The fingerprint of the currently scoped component. Will be a unique id across the entire app.

#### name - (String)

##### Description

The custom name of the currently scoped component, as provided in the editor application.

#### customFields - (Object)

##### Description

The custom fields of the currently scoped component.

#### displayProperties - (Object)

##### Description

The display properties of the currently scoped component.

Pass-through Props
------------------

In order to simplify re-use of functional React components, any attributes provided to the `<Context>` component will be passed through to the child function. However, be careful with using this pattern, as in the event of conflict it will override the context properties you are trying to access.

##### Example

    import React from 'react'
    
    import Context from 'fusion:context'
    
    const ArcSiteComponent = ({ arcSite, label }) => <div>{label}: {arcSite}</div>
    
    const MyFeatureComponent = (props) =>
      <Context label='arc-site'>
        {ArcSiteComponent}
      </Context>
    
    export default MyFeatureComponent