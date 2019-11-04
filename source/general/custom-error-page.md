
Using Custom Error Pages
========================

Implementation
--------------

If during rendering an error occurs fusion will by default print out the error. While this behavior is very useful to us during site buildout in production we will want to make these errors more user friendly. To faciliate this Fusion Engine will automatically look for pages at the path `/error/{statusCode}` and if found will serve these in place of the default error message.

Example
-------

We want to have a 404 page that displays a "Sorry Not Found" message. To do this we would create and publish a page from the PageBuilder Editor called `/error/404` and would add to it our standard features (header, footer, etc) as well as a feature that displays our not found message. Now, when a user comes to a url that results in a 404 they will see our nice 404 page instead of a fusion error message.

Common status codes
-------------------

Here are status codes that fusion can serve which you may want custom error pages for.

Status Code

Meaning

404

Not Found

500

Internal Server Error