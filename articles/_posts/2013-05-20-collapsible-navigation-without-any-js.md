---
layout: article
category: articles
title: Collapsible navigation without any JS
author: dans
abstract: Semantico labs is built using <a href="http://twitter.github.io/bootstrap/">Bootstrap</a> which includes a neat responsive collapsible nav. We use the <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/:target">:target</a> pseudo-class to expand and contract it without using JavaScript
---

When the viewport width is small enough, the links in the nav bar are hidden and a button to expand the list is shown. Bootstrap includes a [jQuery plugin](http://twitter.github.io/bootstrap/javascript.html#collapse) to provide the interaction needed when you click the nav expander button and the nav items animate into view.

On labs we are trying to keep the JavaScript to a minimum. We don’t even include jQuery so using the plugin was out of the question. Instead we use the ```:target``` CSS pseudo-class to achieve the same thing without using any JavaScript.

The ```:target```  pseudo-class represents the unique element, if any, with an ID matching the hash part of the URL of the document. So ```http://example.com/#some-id``` would match the imaginary ```<div id="some-id"></div>```.

We include an ID on the body element of “expanded” and include a ```.btn.btn-navbar``` link to  ```#expanded```. This means we can change any child of the body using the CSS selector ```#expanded:target```. The collapsible nav includes a height of zero by default so we just need to update the ```.nav-collapse``` rule:

<script src="https://gist.github.com/danshearmur/5611968.js"> </script>

This lets you expand the nav but provides no easy way of contracting it (other than the back button). We need to introduce a second ```.btn-navbar``` linking to ```#```. We can again use ```:target``` to show and hide the correct buttons

<script src="https://gist.github.com/danshearmur/5611996.js"> </script>

The main drawback of this approach is that it will add to your browser history. Instead of clicking back and going to the previous page it will scroll to the top of the current page and contract the nav, requiring you to hit back again to actually go to the last page. This isn’t really a problem for us on labs - more an inconvenience.

[Browser support for ```:target```](http://caniuse.com/#search=target) is pretty similar to [media queries](http://caniuse.com/#feat=css-mediaqueries) which are used to hide the nav in the first place.
