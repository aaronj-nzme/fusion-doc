Consumer API
============

The `Consumer` is a higher-order function that can be used to ["decorate"](https://www.sitepoint.com/javascript-decorators-what-they-are/) Fusion components and provide them with useful [props](#props) and [instance methods](#instance-methods) via React's [Context API](https://reactjs.org/docs/context.html). The `Consumer` function can wrap any component type, although it is most typically used for [Features](./api-feature.md). It can be used for both class-based components and functional components to provide `props`, however functional components will be unable to use the [instance methods](#instance-methods) provided by `Consumer`.

Implementation
--------------

The `Consumer` function is imported from the `fusion:consumer` namespace. It can be invoked as a function or using decorator syntax; both produce the same result.

##### Example

_Decorator Syntax_

    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class MyComponent extends Component {
      ...
    }
    
    export default MyComponent
    

_Function Syntax_

    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    class MyComponent extends Component {
      ...
    }
    
    export default Consumer(MyComponent)
    

Props
-----

`Consumer` components will be provided with all [<code>Context</code> props](./api-context.md#Props), along with the following:

### editableContent() (Function)

##### Description

A function that adds the necessary attributes to make the element's content "editable". Use this method to allow PageBuilder editors to change individual ANS content nodes before rendering. This function can be run on a single content node, or on multiple labeled content nodes.

#### Parameters

_Single Node Syntax_

`editableContent(contentObj, contentKeyPath)`

*   `contentObj` (_ANS Object_): The root ANS content object that the `contentKeyPath` should pull from.
*   `contentKeyPath` (_String_): The 'key path' within the ANS object to find the content node that should be editable.

_Multi-Node Syntax_

`editableContent(contentObj, contentNodeMap)`

*   `contentObj` (_ANS Object_): The root ANS content object that the `contentKeyPath` should pull from.
*   `contentNodeMap` (_Object_): A map of labels (acting as keys) to their corresponding `contentKeyPath` values.
    *   `contentNodeMap.{contentLabel}` (_String_): Here, `{contentLabel}` represents the name of the property this content node represents. This label will be capitalized and displayed to an editor in the UI. The value of the key is a `contentKeyPath` string, as in the "single node" syntax above.

#### Return

This function returns 3 attributes to be added to the target element. Fusion uses these attributes to identify the element and apply the content edits.

*   `contentEditable`: An HTML attribute denoting that this element is editable in the browser.
*   `data-content-editable`: An attribute representing the ID of the piece of content and property to be mapped in the ANS document.
*   `data-feature`: Returns the Feature ID of the feature this method is added to.

#### Example

_Single Node Content Editable_

    /*  /components/features/global/editable_headline.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class EditableHeadline extends Component {
      ...
      render() {
        // This `content` would come from a content fetch performed elsewhere in this component
        const { content } = this.state
    
        return (
          <h1 {...this.props.editableContent(content, 'headlines.basic')}>
            {content.headlines.basic}
          </h1>
        )
      }
    }
    
    export default EditableHeadline
    

_Multi-Node Content Editables_

    /*  /components/features/global/editable_headline.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class EditableHeadline extends Component {
      ...
      render() {
        // This `content` would come from a content fetch performed elsewhere in this component
        const { content } = this.state
    
        return (
          <>
            <h1 {...this.props.editableContent(content, {
              headline: 'headlines.basic',
              subheadlines: 'subheadlines.basic',
              description: 'description.basic',
            })}
            >
              {content && content.headlines ? content.headlines.basic : ''}
            </h1>
            <h3>
              {content && content.subheadlines ? content.subheadlines.basic : ''}
            </h3>
            <p>
              {content && content.description ? content.description.basic : ''}
            </p>
          </>
        )
      }
    }
    
    export default EditableHeadline
    

Instance Methods
----------------

### addEventListener() - (Function)

##### Description

This method adds an event listener to a Fusion component that will respond to events of the specified `eventName` by invoking the specified `eventHandler`. Events are dispatched by invoking [<code>dispatchEvent</code>](#dispatchEvent) in other Fusion components. Listeners can be removed by the [<code>removeEventListener</code>](#removeEventListener) method.

##### Parameters

`addEventListener(eventName, eventHandler)`

*   `eventName` (_String_): The name of the event to subscribe to.
*   `eventHandler(payload)` (_Function_): The function that will handle the event when it is triggered. This function receives the event's payload as its only argument.

##### Return

This method returns `undefined`; its effect is to 'subscribe' the event handler to the appropriate event.

##### Example

    /*  /components/features/utils/error-message.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class ErrorMessage extends Component {
      ...
      componentDidMount () {
        this.addEventListener('errorOccurred', (error) => {
          const errorMsg = error && error.message ? error.message : 'Something went wrong'
          this.setState({ message: errorMsg })
        })
      }
      ...
    }
    
    export default ErrorMessage
    

### dispatchEvent() - (Function)

##### Description

This method dispatches an event from a Fusion component of the specified `eventName` with an arbitrary `payload` to be received by another component's event handling function (which gets subscribed via the [<code>addEventListener</code>](#addEventListener) method).

##### Parameters

`dispatchEvent(eventName, payload)`

*   `eventName` (_String_): The name of the event to dispatch, which listeners can subscribe to.
*   \[`payload`\] (_?_): An arbitrary payload attached to the event, for the handler to use in processing.

##### Return

This method returns `undefined`; its effect is to dispatch the event to each subscriber.

##### Example

    /*  /components/features/weather/weather-lookup.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class WeatherLookup extends Component {
      ...
      handleFormInput(value) {
        if (!value || value.length < 5) {
          this.dispatchEvent('errorOccurred', {
            val: value,
            message: 'Zip code must be at least 5 characters long.'
          })
        }
        ...
      }
      ...
    }
    
    export default WeatherLookup
    

### fetchContent() - (Function)

##### Description

The `fetchContent` method is second-level syntactic sugar for using both [getContent](#getContent) and React's `setState` together. It takes a map whose keys are the names of content to be stored in the component's `state` (using `setState`), and the values are configuration options used to fetch content from a content source (using `getContent`). `fetchContent` will then fetch the content using the content configuration and set it on the component's `state` using the key names in `contentConfigMap`.

##### Parameters

`fetchContent(contentConfigMap)`

*   `contentConfigMap` (_Object_): An object whose keys are the names of content to be stored in the component's `state`, and the values are configuration objects idential to those of the [<code>getContent</code>](#getContent) parameters.
    *   `contentConfigMap.{contentKey}` (_Object_): Here, `{contentKey}` represents the name of a property the developer chooses to set on the component's `state` object. Multiple `{contentKey}` objects can exist on the same `contentConfigMap` object.
        *   `contentConfigMap.{contentKey}.source` (_String_): See `source` parameter in [<code>getContent</code>](#getContent) method.
        *   `contentConfigMap.{contentKey}.query` (_Object_): See `query` parameter in [<code>getContent</code>](#getContent) method.
        *   \[`contentConfigMap.{contentKey}.filter`\] (_String_): See `filter` parameter in [<code>getContent</code>](#getContent) method.
        *   \[`contentConfigMap.{contentKey}.inherit`\] (_Boolean_): See `inherit` parameter in [<code>getContent</code>](#getContent) method.

##### Return

This method returns `undefined`; its effect is to set the `state` properties listed in the `contentConfigMap` with the approprite values.

##### Example

    /* /components/features/homepage/topics.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class Topics extends React.Component {
      constructor (props) {
        super(props)
    
        this.fetchContent({
          topics: {
            source: 'content-feed',
            query: { feedType: 'taxonomy.tags.slug', feedParam: '*', limit: 5, offset: 0, order: 'display_date:desc' },
            filter: '{ headline }'
          }
        })
      }
    
      render () {
        return (
          <ul>
            {this.state.topics.map(topic =>
              <li>{topic.headline}</li>
            )}
          </ul>
        )
      }
    }
    
    export default Topics
    

### getContent() - (Function)

##### Description

The `getContent` method will fetch content, both on the server and the client, from a content source (identified by the `sourceName` argument) defined in the bundle.

##### Parameters

For syntactic sugar, there are 2 ways to invoke the `getContent` method: with the arguments expanded and passed individually, or as keys of an object.

_Expanded Syntax_

`getContent(sourceName, query, [filter], [inherit])`

*   `sourceName` (_String_): The name of the content source from which you want to fetch. This content source must be configured in your bundle.
*   `query` (_Object_): This will depend on the definition of the content source, but will be an object containing key/value pairs used to uniquely identify the piece of content you want to fetch.
*   \[`filter`\] (_String_): A GraphQL query string that will be applied to the resultant data to minimize the payload size. This is beneficial for both client-side and server-side fetching, as server-side fetched data must be included in the final HTML rendering to prevent content flashing.
*   \[`inherit`\] (_Boolean_): A dynamic boolean to determine if `globalContent` should be used to override the config settings provided. If this value is `true`, the `globalContent` will be returned in both the `cached` property and as the resolution of `fetched`.

_Object Syntax_

`getContent(options)`

*   `options` (_Object_): An object containing the following properties:
    *   `options.sourceName` (_String_): See `sourceName` parameter above.
    *   `options.query` (_Object_): See `query` parameter above.
    *   \[`options.filter`\] (_String_): See `filter` parameter above.
    *   \[`options.inherit`\] (_Boolean_): See `inherit` parameter above.

##### Return

An object with 2 keys: `{ cached, fetched }`. `cached` will be an object containing data already pre-fetched synchronously on the server from the content source. `fetched` will be a Promise that resolves to an object containing newly fetched data from the content source.

##### Example

    /*  /components/features/weather/forecast.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class WeatherForecast extends Component {
    
      constructor() {
        super(props)
        this.state = { forecast: null }
      }
    
      componentDidMount () {
        navigator.geolocation.getCurrentPosition((location) => {
          // If we get the user's location, call getContent to fetch data from the DarkSky API
          const { fetched } = this.getContent({
            // Specifying the `dark-sky` content source
            sourceName: 'dark-sky',
            // `query` object needs `lat` and `lng` arguments to query the DarkSky API
            query: { lat: location.coords.latitude, lng: location.coords.longitude },
            // GraphQL filter so we get only the data we need
            filter: '{ daily { summary }}'
          })
    
          // Use the `fetched` Promise to get our response and set the forecast info in the component's state
          fetched.then(response => {
            this.setState({ forecast: response.daily.summary })
          })
        })
      }
    
      render() {
        const { forecast } = this.state
        return forecast ? (<p>{forecast}</p>) : null
      }
    }
    
    export default WeatherForecast
    

### removeEventListener() - (Function)

##### Description

This method 'unsubscribes' the specified event handling function (`eventHandler`) from the `eventName` specified. The `eventHandler` must be a reference to the exact function instance that was added via [<code>addEventListener</code>](#addEventListener), not a copy.

##### Parameters

`removeEventListener(eventName, eventHandler)`

*   `eventName` (_String_): The name of the event to unsubscribe the handler from.
*   `eventHandler` (_Function_): A reference to the exact instance of the handler function that was previously added.

##### Return

This method returns `undefined`; its effect is to 'unsubscribe' the event handler from the appropriate event.

##### Example

    /*  /components/features/utils/error-message.jsx  */
    
    import Consumer from 'fusion:consumer'
    import React, { Component } from 'react'
    
    @Consumer
    class ErrorMessage extends Component {
      handleErrorMsg (error) {
        const errorMsg = error && error.message ? error.message : 'Something went wrong'
          this.setState({ message: errorMsg })
        })
      }
    
      componentDidMount () {
        this.addEventListener('errorOccurred', this.handleErrorMsg.bind(this))
      }
    
      componentWillUnmount () {
        this.removeEventListener('errorOccurred', this.handleErrorMsg)
      }
      ...
    }
    
    export default ErrorMessage