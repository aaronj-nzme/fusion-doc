Plugins API
===========

Plugins offer the ability for Feature Pack developers to add a specialized user interface for setting custom field values. Plugins can help editors set complex values when the default behavior of a custom field input isn't enough. Good examples of plugins could include a color picker for selecting hexadecimal color values, a WYSIWYG editor for creating HTML snippets, or a calculator for... calculating.

Implementation
--------------

Plugins in Fusion are implemented as HTML web pages that get included as iframes in the PageBuilder editor. Each HTML document must include a JavaScript method named `initializePlugin` set on the global `window` object. This method is responsible for setting the initial state of the plugin UI, and will be invoked by PageBuilder when a user opens the proper custom field's plugin.

##### Naming

A Plugin is expected to be stored and named in the following format:

*   `/resources/plugins/{pluginName}/index.html`

> The `{pluginName}` portion of the filepath must correspond exactly to the name of the plugin that is passed as the `formPlugin` value for the custom field.

Plugins can also have additional files and folders inside of their respective `pluginName` directories. This may include JavaScript, CSS or other resource dependencies of the plugin.

It is common for plugins to require a build process in order to compile higher-level JavaScript into websafe (ES5) JavaScript. In this case, it is recommended that the _source code_ for the plugin be stored and named in the following format:

*   `/plugins/{pluginName}/*`

> The `*` portion of the filepath represents any source files needed to generate the plugin's compiled code (including HTML).

The source code should contain some build process (typically a webpack config file) that builds a compiled version of the plugin's code into the corresponding `/resources/plugins/{pluginName}` directory.

##### Example

    /*  /resources/plugins/color-picker/index.html  */
    
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8" />
      <title>Plugin</title>
    </head>
    <body>
      <div class="color" data-color="red"></div>
      <div class="color" data-color="white"></div>
      <div class="color" data-color="blue"></div>
    
      <button id="save">Save</button>
      <button id="close">Close</button>
    
      <script>
        window.initializePlugin = function({ field, initVal, onClose, onSave }) {
          let selectedColor = initVal || ''
    
          document.querySelectorAll('.color').forEach(el => {
            el.addEventListener('click', () => {
              selectedColor = el.getAttribute('data-color')
            })
          })
    
          document.getElementById('save').addEventListener('click', () => {
            onSave(selectedColor)
          })
    
          document.getElementById('close').addEventListener('click', onClose)
        }
      </script>
    </body>
    </html>
    

Required Methods
----------------

### window.initializePlugin() (Function)

##### Description

The `initializePlugin` method is required in order for a plugin to be able to interface with PageBuilder by receiving an initial value of the plugin or data about the custom field it is set on, or for the plugin to save a value or close itself. This method must be defined on the global `window` object of the HTML document.

##### Arguments

`initializePlugin(pluginData)`

*   `pluginData` (_Object_): An object containing the data passed by PaegBuilder to the plugin
    *   `pluginData.field` (_Object_): Data about the custom field this plugin is attached to
    *   `pluginData.initVal` (_?_): The initial value of the custom field
    *   `pluginData.onClose()` (_Function_): A callback function to be invoked by the plugin when it should be closed
    *   `pluginData.onSave(value)` (_Function_): A callback function to be invoked by the plugin when a value should be saved. The sole argument to be passed is the value to be saved.

##### Example

See "Implementation" section example above.

Options
-------

### window.pluginOptions (Object)

##### Description

The `pluginOptions` object can be used to set the height and width of the plugin window. It must be set on the global `window` object, similar to the `initializePlugin` method.

##### Keys

*   `pluginOptions.height` (_String_): The CSS value of the desired height of the plugin window.
*   `pluginOptions.width` (_String_): The CSS value of the desired width of the plugin window.

##### Example

    /*  /resources/plugins/color-picker/index.html  */
    ...
    <script>
      window.pluginOptions = {
        height: '300px',
        width: '400px'
      }
      ...
    </script>