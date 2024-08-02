# Document Object Model (DOM)

DOM - a set of APIs for controlling HTML and styling information that makes heavy use of the _Document_ object.

Prerequsites: HTML, CSS, JS - including JS objects;
Objective: To gain familiarity with the core DOM APIs, and other APIs.

Browsers have moving parts many of them can't be controlled or manuipulated by web developer using JS. Browsers
are locked down for centring around secutiry (accesses etc.). Despite the limitations, Web APIs still give access
to lot of functionality.There are a few really obvious bits you'll reference regularly in your code — consider
the following diagram represents the main parts of a browser directly involved in viewing web pages:

![Browser Structure](imgs/BrowserStructure.png)

<img src="imgs/BrowserStructure.png" alt="Browser Structure" width="500" height="300">

- The _Navigator_ represents the state and identity of the browser (i.e. the user-agent) as it exists on the web.
  In JS, this is represented by the _Navigator_ object. You can use this object to retrieve things like: - user's preferred language; - a media stream from the user's webcam, etc.

- The _Window_ is a browser tab that a web page is loaded into; this is represented in JS by the _Window_ object.
  Using methods available on this objet you can do things like return the widnow`s size (like Window.innerWidth and
  Widnow.innerHeight), manipulate the document loaded into that window, store data specific to that document on the
  client-side (for example using a local database or other storage mechanism), attach an event handler to the current
  window, and more.

- The _Document_ (represented by the DOM in browsers) is the actual page loaded into the window, and is represented
  in JS by the _Document_ object. you can use this object to return and manipulate information on the HTML and CSS
  that comprises the document, for exmaple get a reference to an element in the DOM, change its text content, apply new
  styles to it, create new elements and add them to the current element as childern, or even delete it altogether.
