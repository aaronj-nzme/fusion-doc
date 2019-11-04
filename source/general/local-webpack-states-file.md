Locally Generated Webpack Stats File
==========================================

If you're a feature pack developer and you're curious about the composition of the Webpack bundles generated for your pages, Fusion is set up to generate a JSON file of webpack stats that can be ingested by a Webpack bundle analyzer tool.

Note that as of the time of writing this functionality is only available when running Fusion locally.

Finding the stats JSON file
---------------------------

You'll need to know two values before you can find the stats JSON file:

1.  Page or template ID for the page with the bundle you're concerning yourself with.
2.  Output type name for the bundle you want to analyze.

### Page/Template ID

The page/template ID can be most easily found through the admin panel if you go the edit view for the page/template.

The url for the edit view should look something like this:

`http://localhost/pf/admin/app/edit/editor.html?p=AAAAAA&v=BBBBBB`

The page ID can be found in the value set for the `p` query param. In this case the value is `AAAAAA`.

### Output Type Name

The output type is usually `default` unless you have some other output type in mind.

### Stats File Location

The stats JSON file can be found in two places:

1.  In your computer's feature pack repo at `.fusion/dist/page/<PAGE_ID>/<OUTPUT_TYPE>.stats.json`.
2.  Served from localhost: `<CONTEXT_PATH>/dist/page/<PAGE_ID>/<OUTPUT_TYPE>.stats.json`.

For example, the `.stats.json` file for a page with an ID of `AAAAAA` and an output type of `default` can be found in your feature pack repo at `.fusion/dist/page/AAAAAA/default.stats.json` and served from localhost at `http://localhost/pf/dist/page/AAAAAA/default.stats.json`.

Analysis
--------

There a number of free web-based tools that ingest `.stats.json` files and display useful data about your bundle:

*   [https://chrisbateman.github.io/webpack-visualizer/](https://chrisbateman.github.io/webpack-visualizer/)
*   [http://webpack.github.io/analyse/](http://webpack.github.io/analyse/)