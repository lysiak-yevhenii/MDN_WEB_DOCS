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

The _Node.appendChild()_ - adds a node to the end of the list of children of a specified parent node. In our case parent node is `<select>`
and childs are `<img>` and `<p>` elements, so _Node.appendChild(para)_ will add paragraph to the end of this tree level.

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

![Final DOM](imgs/FinalDOM.png)

# Moving and removing elements

## Moving

_sect.appendChild(linkPara)_ - will move the paragraph with link inside it to the bottom of the section.
you might have thought it would make a second copy of it, but not the case - linkPara is reference to the
one and only copy of that paragraph.
_Node.cloneNode()_ - if you want to make a copy, you need to use that method.

## Removing

_sect.removeChild(linkPare)_ - if you have a reference to the node, you can easily remove node from parent.
_linkPara.remove()_ - when you want to remove node based only in a reference to itself (this method not
supported by older browsers). But old browsers support next construction:
to delete through the parent ref. _linkPara.parentNode.removeChild(linkPara)_.

# Manipulating styles

There is variety of approacher to get and manipulate the styles. One of them is to use _Document.stylesheets_,
which returns an array-like object with _CSSStyleSheet_ objects. You can then add/remove styles as whished.

Add inline styles directly onto elements you want to dynamically style. To do so use _HTMLElement.style_ property,
which contains inline styling information for each element in the document. You can set properties of this object to
directly update element styles:

```
para.style.color = 'white'
para.style.backgroundColor = 'black'
para.style.padding = '10px'
para.style.width = '100px'
para.style.textAlign = 'center'
```

If you apply this way the style. It will looks like an inline style:

```
<p
  style="color: white; background-color: black; padding: 10px; width: 250px; text-align: center;">
  We hope you enjoyed the ride.
</p>
```

Another common way to dynamically manipulate styles on your document is usage of attribute attachment.
Let's imagine you have defined class in CSS called 'highlight' with style defined there. You can use
next: _para.setAttribute('class','highlight')_ and it will add class to the para element, and as result
it will apply highligh style.

# Task

In this challenge we want to make a simple shopping list example that allows you to dynamically add items to the list using a form input and button. When you add an item to the input and press the button:

- The item should appear in the list.
- Each item should be given a button that can be pressed to delete that item off the list.
- The input should be emptied and focused ready for you to enter another item.
