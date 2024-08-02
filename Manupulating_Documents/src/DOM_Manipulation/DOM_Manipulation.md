# Basic DOM manipulation

```
<!DOCTYPE html>
<html lang="en-US">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple DOM example</title>
  </head>
  <body>
      <section>
        <img src="dinosaur.png" alt="A red Tyrannosaurus Rex: A two legged dinosaur standing upright like a human, with small arms, and a large head with lots of sharp teeth.">
        <p>Here we will add a link to the <a href="https://www.mozilla.org/">Mozilla homepage</a></p>
      </section>
  </body>
</html>
```

## Here is HTML we will work with.

1. We need to add <script></script> element about the closing </body> tag.
2. To manipulate an element inside the DOM, you first need to select it and store a reference to it inside a variable.
   Inside the script element, add the following line:

```
const link = document.querySelector("a");
```

3. Now we have the element reference stored in the variable _link_, we can manipulate it using properites and methods
   available to it (these are defined on interface like HTMLAnchorElement in the case of <a> element, its more general
   parent interface HTMLElement, and Node - which represents all nodes in a DOM). Let's change the text inside the link
   by updating the value of the _Node.textContent_ property. Add following line below the previous one:

```
link.textContent = 'Mazilla Developer Network'
```

4. We should also change the URL the link is pointing to, so that it doesn't go to the wrong place when it is clicked on.
   Add the following line, again at the bottom:

```
link.href = 'https://developer.mozilla.org'
```

_Document.querySelector()_ - is the recommended modern approach. It's convenient becaise ot allows you to select elements
using CSS selectors. The above _querySelector()_ call will match the first <a> element that apperats in the document. If you
wanted to match and fo things to multiple elements, you could use _Document.querySelectorAll()_, which matches every element
in the document that matches the selector, and stores references to them in an array - like object called a NodeList. In other
words the return of _Document.querySelectorAll()_ is _NodeList_.

I think is good to directly move to deep explanation what querySelector() do. It returns the first Element within the document
that matches the specified CSS selector. The CSS selectors module provides us with more than 60 selectors and five combinators.
For the beginning main are: class and id selectors:

- To select classes you can use next argument value for querySelector(".hereClassName").
- To select element by Id use next argument value for querySelector("#hereId").

## Creating and placing nodes

1. Let's start by grabbing a reference to our <selection> element - add the following code at the bottom of your exising script
   (do the same with the other lines too):

```
const sect = document.querySelector('section');
```

2. Let's create new paragraph using _Document.createElement()_ and give it some text content in the same way as before:

```
const para = document.createElement('p');
para.textContent = "We hope you enjoyed the ride.";
```

3. You can now append the new paragraph at the end of the section using _Node.appendChild()_:

```
sect.appendChild(para);
```

_Node.appendChild()_ - adds a node to the end of the list of children of a specified parent node. In our case parent node is <select>
and childs are <img> and <p> elements, so _.appendChild(para)_ will add paragraph to the end of this tree level.

4. Finally let's add a text node to the paragraph the link sits inside, to round off the sentence nicely. We will create
   text node using _Document.createTextNode()_:

```
const text = document.createTextNode(" - the premier source for web development knowledge.",);
```

5. Now we'll grab a reference to the paragraph the link is inside, and append the text node to it:

```
const linkPara = document.querySelector('p');
linkPara.appendChild(text);
```

How the final DOM will looks like:

![Final DOM](Manupulating_Documents/src/DOM_Manipulation/imgs/FinalDOM.png)
