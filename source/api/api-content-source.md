Content Source API
==================

Implementation
--------------

##### Naming

A Content Source is expected to be stored and named in the following format:

*   `/content/sources/{sourceName}.(js|json)`

> This will build a content source whose name is represented by the `{sourceName}` portion of the filepath.

### JavaScript Definition

##### Properties

A Content Source defined in JavaScript should return an object with the following properties:

*   `resolve(query)` (_Function_): A function that, given a query object, returns a URI to fetch content from that will return JSON.
*   `fetch(query)` (_Async Function_): A function that, given a query object, will return JSON. May be async. Content Source should include exactly one of `resolve` or `fetch`.
*   `cache` (_Boolean_): Flag for whether content associated with this source should be cached by Fusion. Default is `true`.
*   `http` (_Boolean_): Flag for whether this content source should be accessible via the HTTP API. If false, the content source is still available to components during server-side rendering. Default is `true`.
*   `params` (_Object_): A map of key/value pairs whose keys are the names of parameters to be used in the `query` object passed to the `resolve` method when invoked, and whose values are the "type" of data that parameter can hold. The available "types" are `text`, `number`, and `site`.
*   \[`schemaName`\] (_String_): The name of a content schema (without the file extension) defined in the `/content/schemas/` directory that this content source corresponds to.
*   \[`transform(json, query)`\] (_Function_): A function that, given the JSON returned from the endpoint defined in the `resolve` function, returns a version of that JSON with some transformation applied to it.
*   \[`filter`\] (_String_): A GraphQL query string that will be applied to the resultant data to minimize the payload size. It can be beneficial to apply this kind of filtering on content source for reducing boilerplate and doing content filtering on `globalContent` values.
*   \[`ttl`\] (_Number_): The number of seconds content fetched from this content source should be cached for. Default is `300` (5 minutes); the `ttl` cannot be set lower than `120` (2 minutes).
*   \[`serveStaleCache`\] (_Boolean_): Flag for whether this content source should serve stale cache items to improve content source resiliency when a content fetch returns an error response (> 400, excluding 404s). Default is true.

##### Example

    /*  /content/sources/content-api.js  */
    
    const resolve = function resolve (query) {
      const requestUri = `/content/v3/stories/?canonical_url=${query.canonical_url || query.uri}`
    
      return (query.hasOwnProperty('published'))
        ? `${requestUri}&published=${query.published}`
        : requestUri
    }
    
    module.exports = {
      resolve,
      schemaName: 'minimal',
      params: {
        canonical_url: 'text',
        published: 'text'
      }
    }
    

### JSON Definition

It's also possible to define a content source in JSON. This use case is mostly to support legacy content configurations that are being ported over in JSON format. Is possible, we recommend that you define your content sources using the JavaScript Definition syntax above, if only so that we can keep credentials that should be encrypted out of the Feature Pack bundle.

##### Properties

A Content Source defined in JSON should have the following properties:

*   `id` (_String_): The name of the content source being defined.
*   `content` (_String_): The name of the content schema that this content source is using. Treat this like the `schemaName` property in the JS definition.
*   `params` (_Array\[Object\]_): An array of objects containing configuration data for each parameter included in this content source. Each object should contain the following properties:
    *   `name` (_String_): The name of the param to be used in the `pattern` section.
    *   `displayName` (_String_): A human readable title for the param to be used in PageBuilder.
    *   `type` (_String_): The "type" of the param. Available options are `text`, `number`, and `site.`
    *   \[`default`\] (_String_): The default value this parameter should contain if it is not available at runtime.
*   `pattern` (_String_): A URI string that is able to interpolate data enumerated by the `params` property with curly braces to contruct the full URL that represents this content source.

##### Example

    {
      "id": "darksky",
      "content": "weather",
      "params": [
        {
          "name": "lat",
          "displayName": "Latitude",
          "type": "number",
          "default": "38.88",
        },
        {
          "name": "lon",
          "displayName": "Longitude",
          "type": "number",
          "default": "-77.00",
        },
      ],
      "pattern": "https://api.darksky.net/forecast/SOME_UNENCRYPTED_API_KEY/{lat},{lon}"
    }