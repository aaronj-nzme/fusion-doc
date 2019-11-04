Local Mocks API
===========

During development, sometimes there's a need to mock resources locally. Anything inside of the `/mocks` folder will be served from the root of your website when developing locally. This functionality exists to make local development more convenient, but beware that mock resources will not be available in production.

Service Workers
---------------

One use case for the `/mocks` folder is to install a [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers) at the root scope of your website. We can mock this functionality by placing the service worker script inside the `/mocks` folder of our feature pack. This allows us to customize and test the service worker locally.

Moving to Production
--------------------

If resources are required for your production website, we recommend that those resources are moved to the `/resources` folder of your feature pack. For resources that must be located at the root of your website, the Arc Delivery team can configure your Content Delivery Network to ensure that specific resources are served from the correct location.