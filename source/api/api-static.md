Static Component API
====================

Fusion `<Static>` is a React component for designating static content that should not be re-rendered in the browser.

The primary use for this component is to preserve any content and event handlers exactly as they exist on first browser render.

Unlike static features (or chains) defined with the [static](./api-feature.md) property, code defined inside a Static component will still be sent to the client. Do not use a Static component to hide credentials.

Implementation
--------------

Example:

    'use strict'
    
    import PropTypes from 'prop-types'
    import React from 'react'
    import Static from 'fusion:static'
    
    const Article = ({ customFields, globalContent }) =>
      <div>
        <h1>{globalContent.title}</h1>
        <Static id={ customFields.id }>
          {globalContent.embeddedHTML}
        </Static>
      </div>
    
    Article.propTypes = {
      customFields: PropTypes.shape({
        id: PropTypes.string.isRequired
      })
    }
    
    export default Article
    

Props
-----

#### id (String)

An `id` attribute that is both unique and consistent (between server and client renders) must be provided to the Static component.

#### htmlOnly (Boolean)

The Static component works by creating a cache of all static components by id before attempting a React client-side render. This cache is then used to hydrate the content of each Static component during the React client-side render. The original DOM elements are generally inserted back into place. This implementation will preserve any event handlers that are attached to the elements. However, in the case that an id cannot be guaranteed to be unique (e.g., an image that may exist in multiple places on a page), only the final instance will be hydrated as the DOM elements will just be moved around the page.

If there are no event handlers expected within the Static component, you may set the `htmlOnly` prop to signal that clones of the original DOM elements are sufficient. In this case, duplicate ids will each get a copy of the original elements, but will have no event handlers.