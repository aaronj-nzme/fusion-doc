Creating a Web API Returning a Non-HTML Content Type
====================================================

Fusion supports non-HTML output types like JSON data and XML. This can be utilized to build web APIs for other usages. Let's build an API for your mobile applications to get a list of recommended sections by editor's pick and their most recent stories.

Goals
-----

*   The mobile API should return a list of sections base on the configuration in PageBuilder Admin.
*   For each configured section, it should contain the section's id, name, and x number of most recent stories (x is configurable).
*   There should be a visual preview for configuring the mobile API in PageBuilder Admin.

Design
------

### JSON output type

Building web APIs using JSON output type is similar to building regular web pages. You need to create an output type and features/chains/layouts. The main difference is you don't create them as React components. All the files must use `.js` file extension, and each rendering method returns a plain JavaScript object. Since usually the payload structure of a web API is either a single object or an array of objects, we can use a single feature to generate the object type payload or a single chain with multiple features to generate the array type payload. This way we can simply create a generic JSON output type and a layout which directly return the first child of the children from the props. Note here, using layout is required due to a known issue in Fusion to receive consistent data structure for the children in JSON output.

This is an entity-relationship diagram for object type payload:

                    ---------------
                    | Output Type |
                    ---------------
                          |
                          |
                      ----------
                      | Layout |
                      ----------
                          |  
                          |
                     -----------
                     | Feature |
                     -----------
    

This is an entity-relationship diagram for array type payload:

                    ---------------
                    | Output Type |
                    ---------------
                          |
                          |
                      ----------
                      | Layout |
                      ----------
                          |  
                          |
                      ---------
                      | Chain |
                      ---------
                          |        
                          |    
                         /|\
                    ------------  
                    | Features |  
                    ------------
    

For this example, we will use this design approach; however, keep in mind this is not the only way you can use. The real design should reflect to your use cases.

### XML output type

The above design approach can be applied to none-JSON content types as well. We will build an XML output type as an illustration. The key to achieve this is the output type must return a string as the final result.

Creating a generic JSON output type
-----------------------------------

This generic JSON output type returns the first child in the `props.children` which is the same data returned from a layout.

    /* /components/output-types/json.js */
    
    const Json = ({ children }) => {
      // Only return the data from the first child.
      return Array.isArray(children) ? children[0] : null
    }
    
    // Specify content type
    Json.contentType = 'application/json'
    
    export default Json
    

Creating a generic api layout
-----------------------------

This generic JSON layout returns the first child in the `props.children` which is the same data returned from the first chain or feature. Since we will create visual preview later, we are using output type naming convention `api/json.js` for different output types.

    /* /components/layouts/api/json.js */
    
    const Api = ({ children }) => {
      // Only return the data from the first child (body)
      return Array.isArray(children) ? children[0] : null
    }
    
    Api.sections = [
      'body'
    ]
    
    export default Api
    

Creating a chain to return array type data
------------------------------------------

We will create visual preview later, so we are using output type naming convention `ApiList/json.js` for different output types.

    /* /components/chains/ApiList/json.js */
    
    const ApiList = ({ children }) => children
    
    export default ApiList
    

Creating a content source to fetch section data
-----------------------------------------------

    /* /content/sources/section.js */
    
    const resolve = ({
      website,
      id
    }) => `/site/v3/website/${website}/section?_id=${id}`
    
    export default {
      resolve,
      params: [
        {
          name: 'website',
          displayName: 'Website',
          type: 'text'
        },
        {
          name: 'id',
          displayName: 'Section ID',
          type: 'text'
        }
      ]
    }
    

Creating a content source to fetch stories by section
-----------------------------------------------------

    /* /content/sources/stories-by-section.js */
    
    const resolve = ({
      website,
      section,
      size = 1,
      offset = 0
    }) => {
      const body = encodeURIComponent(JSON.stringify({
        query: {
          bool: {
            must: [
              {
                term: {
                  type: 'story'
                }
              },
              {
                nested: {
                  path: 'taxonomy.sections',
                  query: {
                    bool: {
                      must: {
                        term: {
                          'taxonomy.sections._id' : section
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }))
      return `/content/v4/search/published/?website=${website}&body=${body}&size=${size}&from=${offset}&sort=publish_date:desc`
    }
    
    export default {
      resolve,
      params: [
        {
          name: 'website',
          displayName: 'Website',
          type: 'text'
        },
        {
          name: 'section',
          displayName: 'Section ID',
          type: 'text'
        },
        {
          name: 'size',
          displayName: 'Site',
          type: 'number'
        },
        {
          name: 'offset',
          displayName: 'Offset',
          type: 'number'
        }
      ]
    }
    

Creating a feature to combine section data and stories
------------------------------------------------------

In this feature, we need to fetch both section and stories data using the content sources we created above and return the desired data structure. Since we will create visual preview later, we are using output type naming convention `Section/json.js` for different output types.

    /* /components/features/api/Section/json.js */
    
    import PropTypes from 'prop-types'
    import Consumer from 'fusion:consumer'
    
    const WEBSITE_ID = 'your_website_id'
    
    @Consumer
    class Section {
      constructor (props) {
        this.props = props
        const {
          customFields: {
            section,
            size = 3
          } = {}
        } = props
    
        this.fetchContent({
          section: {
            source: 'section',
            query: {
              website: WEBSITE_ID,
              id: section
            }
          },
          result: {
            source: 'stories-by-section',
            query: {
              website: WEBSITE_ID,
              section,
              size
            }
          }
        })
      }
    
      render () {
        const {
          section,
          result
        } = this.state || {}
    
        if (!section || !result) {
          return null
        }
    
        return {
          id: section._id,
          name: section.name,
          stories: result.content_elements
        }
      }
    }
    
    Section.propTypes = {
      customFields: PropTypes.shape({
        section: PropTypes.string.tag({
          label: 'Section ID'
        }).isRequired,
        size: PropTypes.number.tag({
          label: '# of Stories',
          defaultValue: 3
        })
      })
    }
    
    export default Section
    

Creating corresponding components for default output for visual preview
-----------------------------------------------------------------------

With all the components created above, you should be able to build the mobile API in PageBuilder Admin as a page to serve recommended sections and their stories using `outputType=json`. However the editing process might not be convenient as it's just a bunch of JSON data if using JSON output for preview.

    /* /components/layouts/api/default.jsx */
    
    import React from 'react'
    import PropTypes from 'prop-types'
    
    const Api = ({ children }) => {
      return (
        <>
          {children}
        </>
      )
    }
    
    Api.sections = [
      'body'
    ]
    
    Api.propTypes = {
      children: PropTypes.array
    }
    
    export default Api
    

    /* /components/chains/ApiList/default.jsx */
    
    import React from 'react'
    import PropTypes from 'prop-types'
    
    const ApiList = ({ children }) => <div>{children}</div>
    
    ApiList.propTypes = {
      children: PropTypes.any
    }
    
    export default ApiList
    

    /* /components/features/api/Section/default.jsx */
    
    import React, { Component } from 'react'
    import PropTypes from 'prop-types'
    import Consumer from 'fusion:consumer'
    
    const WEBSITE_ID = 'your_website_id'
    
    @Consumer
    class Section extends Component {
      constructor (props) {
        super(props)
        const {
          customFields: {
            section,
            size = 3
          } = {}
        } = props
    
        this.fetchContent({
          section: {
            source: 'section',
            query: {
              website: WEBSITE_ID,
              id: section
            }
          },
          result: {
            source: 'stories-by-section',
            query: {
              website: WEBSITE_ID,
              section,
              size
            }
          }
        })
      }
    
      render () {
        const {
          section,
          result
        } = this.state || {}
    
        if (!section || !result) {
          return null
        }
    
        const stories = result.content_elements || []
    
        return (
          <div>
            <h3>Section {section.name} ({section._id})</h3>
            <ol>
              {
                stories.map((story) => (
                  <li key={story._id}>{story.headlines.basic}</li>
                ))
              }
            </ol>
          </div>
        )
      }
    }
    
    Section.propTypes = {
      customFields: PropTypes.shape({
        section: PropTypes.string.tag({
          label: 'Section ID'
        }).isRequired,
        size: PropTypes.number.tag({
          label: '# of Stories',
          defaultValue: 3
        })
      })
    }
    
    export default Section
    

Now check the page you created earlier in PageBuilder Admin, you should be able to see the visual presentation for each section and click through for modification.

Creating corresponding components for XML output type
-----------------------------------------------------

Since we need to return a string for the XML output type, we will use the [xmlbuilder npm module](https://www.npmjs.com/package/xmlbuilder) to build the DOM tree from a plain JavaScript object and convert it to an XML string. To simply the object conversion, the feature, chain, and layout should return objects using the [structure](https://github.com/oozcitak/xmlbuilder-js/wiki/Conversion-From-Object). You can also use other XML builders if you have a different preference.

    /* /components/output-types/xml.js */
    
    import xmlbuilder from 'xmlbuilder'
    
    const Xml = ({ children }) => {
      // Only return the data from the first child.
      console.log(children[0])
      return xmlbuilder.create({
        service: children[0] || ''
      }, {
        separateArrayItems: true
      }).end({
        pretty: true
      })
    }
    
    // XML content type
    Xml.contentType = 'application/xml'
    
    export default Xml
    

    /* /components/layouts/api/xml.jsx */
    
    const Api = ({ children }) => {
      // Only return the data from the first child (body)
      return Array.isArray(children) ? children[0] : null
    }
    
    Api.sections = [
      'body'
    ]
    
    export default Api
    

    /* /components/chains/ApiList/xml.jsx */
    
    const ApiList = ({ children }) => {
      // Remove null results
      return children.filter((child) => !!child)
    }
    
    export default ApiList
    

    /* /components/features/api/Section/xml.jsx */
    
    import PropTypes from 'prop-types'
    import Consumer from 'fusion:consumer'
    import get from 'lodash.get'
    
    const WEBSITE_ID = 'your_website_id'
    
    @Consumer
    class Section {
      constructor (props) {
        this.props = props
        const {
          customFields: {
            section,
            size = 3
          } = {}
        } = props
    
        this.fetchContent({
          section: {
            source: 'section',
            query: {
              website: WEBSITE_ID,
              id: section
            }
          },
          result: {
            source: 'stories-by-section',
            query: {
              website: WEBSITE_ID,
              section,
              size
            }
          }
        })
      }
    
      render () {
        const {
          section,
          result
        } = this.state || {}
    
        if (!section || !result) {
          return null
        }
    
        // For the example, we will only include id, headline, and description for each story
        return {
          section: {
            id: section._id,
            name: section.name,
            stories: result.content_elements.map((story) => ({
              story: {
                id: story._id,
                headline: get(story, 'headlines.basic', ''),
                description: get(story, 'description.basic', '')
              }
            }))
          }
        }
      }
    }
    
    Section.propTypes = {
      customFields: PropTypes.shape({
        section: PropTypes.string.tag({
          label: 'Section ID'
        }).isRequired,
        size: PropTypes.number.tag({
          label: '# of Stories',
          defaultValue: 3
        })
      })
    }
    
    export default Section
    

After you create all these components, you can use `outputType=xml` to view the XML payload.