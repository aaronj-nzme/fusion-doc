Using Service Workers with Fusion
=================================

Fusion developers often find themselves wanting to implement the latest and greatest PWA technologies in their Fusions applications. This document covers [service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API), which act as proxies between your apps and the external services they use.

We've provided some specific behavior in Fusion that is specifically intended to support serving up service worker scripts. Service worker scripts kept in the `resources` directory of your Fusion feature pack are automatically stored and served directly from S3, speeding up the delivery of those script assets.

Additionally, we've set up the server to simplify the work needed to pull down those service worker scripts. If a request is made for a `.js` file from the root domain without an `outputType` query param (ex. `https://www.yourcontextroot.com/sw.js` or just `/sw.js`), our server will automatically look for for the file that is placed at `resources/sw.js` in your repo and serve that file. You don't need to concern yourself with figuring out the correct path to get that `.js` file to the front-end.

For a service worker script located in `resources/sw.js` in your repo all you'll need to do to install and register that service worker script is to call something like `navigator.serviceWorker.register('/sw.js')` and it will automatically get pulled down from S3.

**Note**: As of the time of writing this special routing has only been created for files ending in `.js` without an `outputType` query param. No such similar routing has been done for any other type of file in `resources`.
