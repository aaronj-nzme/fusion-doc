Including Static Features Outside of Fusion
===========================================

In certain cases you may need to include a Fusion component on a non-Fusion rendered site or page. A common use case for including a Fusion component in non-Fusion environments may include rendering a shared header and/or footer in a different site or application that should have similar features as your Fusion application. The other application will, presumably, have its own rendering engine that may not be written in React or even in JavaScript (gasp!). Instead of trying to render the React component's source directly in this application, we will instead use a work-around that allows Fusion to render the relevant HTML fragment from an API endpoint, and then the non-Fusion application can fetch that HTML and include it directly in its server side payload.

It's worth noting that this method of generating an HTML fragment to be included elsewhere won't support components that need client-side rendering or interactivity - only components that can be rendered statically on the server. So if your shared component won't work without client-side JavaScript, it's probably not a good candidate for this implementation.

In this example, we'll assume you want to use a shared `global/header.jsx` feature from your Fusion app in a different non-Fusion app. We won't define that feature in our code here since it's outside the scope of the article, but we'll assume it's a regular Fusion component that is marked as `.static = true`.

Create an Output Type
---------------------

Since we are rendering an HTML fragment, rather than an entire HTML webpage, we need a custom Output Type for this purpose. We'll create a new Output Type named `ServerSideInclude` for this purpose, but in practice you can name it anything you'd like:

    /*  /components/output-types/server-side-include.jsx  */
    
    import React, { Fragment } from 'react'
    import PropTypes from 'prop-types'
    
    const ServerSideInclude = ({ children }) => {
      return (
        <Fragment>
          {children}
        </Fragment>
      )
    }
    
    ServerSideInclude.propTypes = {
      children: PropTypes.any
    }
    
    ServerSideInclude.contentType = 'text/html'
    
    export default ServerSideInclude
    

This is the simplest possible Output Type we could create - all we are doing is rendering the children of this component into a Fragment.

Replace relative links in CSS
-----------------------------

While the above Output Type will work, it won't have any of the CSS to style the component. In a normal Output Type we could include the CSS with our typical `<CssLinks />` tag; however, that won't work for 2 reasons:

*   we don't want to add an external `<link>` tag to our fragment, we want the CSS defined in a `<style>` tag, but more importantly
*   we need to make all references to relative links in the CSS absolute paths, since this code will be used on a different domain than the domain the assets are served from.

So let's make use of some custom logic and a `<Resource>` tag to get the CSS code and transform it into a `<style>` tag with absolute-path URLs directly:

    /*  /components/output-types/server-side-include.jsx  */
    
    import React, { Fragment } from 'react'
    import PropTypes from 'prop-types'
    
    // Define the rootDomain of the site to be used later. In a real use case, you may have logic here to get this from an environment variable or elsewhere instead of hardcoding it
    const rootDomain = 'https://my-website-domain.com'
    
    const ServerSideInclude = ({ children, Resource, contextPath }) => {
      return (
        <Fragment>
          {/* Using a Resource tag to import the raw styles as `data` */}
          <Resource path={`resources/styles.css`}>
            {({ data }) => {
              if (!data) return null
    
              {/* Using the string `replace` function to replace any CSS URLs with  */}
              const css = data.replace(/url\((\.[^\s<>{}|\\^~)'"`]+)\)/g, (_, uri) => {
                // replace any url that starts with a `..`
                const trimmedUri = uri ? uri.replace(/\.\.\//g, '') : uri
                return `url(${rootDomain}${contextPath}/resources/${trimmedUri})`
              })
              return <style dangerouslySetInnerHTML={{__html: css}} />
            }}
          </Resource>
          }
          {children}
        </Fragment>
      )
    }
    
    ServerSideInclude.propTypes = {
      children: PropTypes.any,
      Resource: PropTypes.func,
      contextPath: PropTypes.string
    }
    
    ServerSideInclude.contentType = 'text/html'
    
    export default ServerSideInclude
    

In the code above, we've used a Regular Expression to replace relative links in our CSS with absolute ones, which should ensure our CSS works anywhere this HTML fragment is rendered. This function may need to be changed to work with your code, but the idea is the same - make relative URLs into absolute ones.

Replace relative links in HTML
------------------------------

We're almost done with this Output Type - but first, we need to do the same thing for our HTML that we did for our CSS. It's possible that in the HTML fragment we're rendering relative links within the page (e.g. to an image or other static asset), which we don't want to do. We need to update these to be absolute URLs to our Fusion domain as well. In order to fix this, we will use the handy `transform` object that is available on Output Types. The `transform` object is a post-render hook that allows us to add a method that takes in the fully-rendered HTML of this output type, and transform it somehow into a different Output Type. The name of the method we define in the `transform` object will become the name of the Output Type we want to use later.

Here's how to use it:

    /*  /components/output-types/server-side-include.jsx  */
    
    import React, { Fragment } from 'react'
    import PropTypes from 'prop-types'
    
    const rootDomain = 'https://my-website-domain.com'
    
    // Add a method to replace relative `href` and `src` URLs with absolute versions of the URL
    const relativeToAbsolutePaths = (html) => html
      .replace(
        /(href|src)=(["'])(?!(https?:)?\/\/)(.*?)\2/g,
        `$1=$2${rootDomain}$4$2`
      )
    
    // Output type definition stays the same...
    ...
    
    // Add our transform method to replace relative links with absolute ones
    ServerSideInclude.transform = {
      ssi ({ data }) {
        return {
          contentType: 'text/html',
          data: relativeToAbsolutePaths(data)
        }
      }
    }
    
    export default ServerSideInclude
    

In our `transform` object we've defined an `ssi()` method which is responsible for returning the proper `contentType` (`text/html` in this case) and the HTML with updated paths. Now, when we use the `ssi` Output Type to render a component (or set of components) we will get the HTML fragment back with absolute path URLs.

Configure the fragment in PageBuilder
-------------------------------------

Now that we have an Output Type that is configured to render HTML fragments with absolute URLs, all the work we need to do should be in the PageBuilder Editor. First, we'll want to create a page in PageBuilder with a descriptive URL. Since we're rendering a header, let's make the path to this page `/ssi-header`. Next, in the editor view for this page, we'll want to add our `Global Header` feature to the page, which is the component we are actually trying to share. When we publish this page and view it at the proper URL (which should be `/ssi-header?outputType=ssi`), we'll get an HTML fragment that includes the static HTML for our page, along with an embedded `<style>` tag, all with absolute-path URLs for links back to the Fusion instance we're currently running on.

Now in the other application we're running, it's simply a matter of fetching content from this `/ssi-header?outputType=ssi` endpoint and rendering the fragment into the server side payload using whatever programming language you deem fit.