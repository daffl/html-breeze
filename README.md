# Breezy

A fast HTML5 view engine that renders using [virtual-dom](https://github.com/Matt-Esch/virtual-dom) in the browser and strings in NodeJS.

[![Build Status](https://secure.travis-ci.org/daffl/breezy.png)](http://travis-ci.org/daffl/breezy)

## Example

Breezy uses custom HTML elements and attributes to render HTML5 based templates with [Expressions]() as placeholders. For example, the following HTML template:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{site.title}}</title>
</head>
<body>
  <h1>{{name.toUpperCase}}'s image gallery</h1>

  <ul>
    <li for-each="images">
        <img src="{{src}}" alt="{{description}}"
            class="{{first $this ? 'first'}} {{last $this ? 'last'}}">
    </li>
  </ul>
</body>
</html>
```

Rendered with the following data

```js
var data = {
  site: { title: 'My page' },
  name: 'david',
  first: function(image) {
    return this.images.indexOf(image) === 0;
  },
  last: function(image) {
    return this.images.indexOf(image) === this.images.length - 1;
  },
  images: [{
    src: 'http://placehold.it/350x150',
    description: 'The first image'
  }, {
    src: 'http://placehold.it/350x150',
    description: 'Another image'
  }, {
    src: 'http://placehold.it/350x150',
    description: 'The last image'
  }]
};
```

Will result in:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My page</title>
</head>
<body>
<h1>DAVID's image gallery</h1>

<ul>
    <li for-each="images">
        <img src="http://placehold.it/350x150" alt="The first image" class="first ">
    </li><li for-each="images">
        <img src="http://placehold.it/350x150" alt="Another image" class=" ">
    </li><li for-each="images">
        <img src="http://placehold.it/350x150" alt="The last image" class=" last">
    </li>
</ul>
</body>
</html>
```

## Usage

Breezy can be used with NodeJS where it outputs a plain string or in the browser where a [virtual-dom](https://github.com/Matt-Esch/virtual-dom) is created which is then used to quickly update only the parts of the DOM that actually changed.

In the browser you will also get automatically updating templates as your data changes when [Polymer's ObserveJS](https://github.com/Polymer/observe-js) is included.

### NodeJS

To use Breezy programmatically in Node just require it and use `render(content, data)` or `renderFile(path, data)`:

```js
var breezy = require('../lib/breezy');
var data = {
  site: { title: 'My page' },
  name: 'david',
  first: function(image) {
    return this.images.indexOf(image) === 0;
  },
  last: function(image) {
    return this.images.indexOf(image) === this.images.length - 1;
  },
  images: [{
    src: 'images/first.png',
    description: 'The first image'
  }]
};
var html = breezy.renderFile(__dirname + '/public/page.html', data);
console.log(html);

// Or compiled with a template string
var renderer = breezy.compile('<div>{{site.title}} from {{name}}</div>');

console.log(renderer(data));
```

It can also be loaded as a view engine in your [Express](http://expressjs.com/) app:

```js
var express = require('express');
var app = express();
var data = {
  site: { title: 'My page' },
  name: 'david',
  first: function(image) {
    return this.images.indexOf(image) === 0;
  },
  last: function(image) {
    return this.images.indexOf(image) === this.images.length - 1;
  },
  images: [{
    src: 'images/first.png',
    description: 'The first image'
  }]
};

app.set('view engine', 'breezy');
app.set('views', __dirname + '/templates');
app.get('/', function(req, res) {
  res.render('page.html', data);
});

app.listen(3000);
```

### Browser

The easiest way to get Breezy into the browser is via the [Bower](http://bower.io/) package:

> bower install breezy

You can also download the distributable from the latest [release](https://github.com/daffl/breezy/releases). Then include it in your page:

```html
<script src="bower_components/breezy/dist/breezy.js"></script>
```

`dist/breezy.js` can also be loaded as an AMD and CommonJS module. If included without a module loader, the global variable `breezy` will be available.

Breezy has no hard dependencies but if you want your templates to automatically update when the displayed data change, you will also need to include [ObserveJS](https://github.com/Polymer/observe-js):

> bower install observe-js

And in the page as well:

```html
<script src="bower_components/observe-js/src/observe.js"></script>
```

Next we have to supply the template and data that we want to render to `breezy.render`. You can use a string (for example the `.innerHTML` content of a `<script type="text/html>` tag) or create your templates in-page and pass the DOM element that you want to make live. All together it looks like this:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My image gallery</title>
</head>
<body>
  <div id="application">
    <h1>{{name.toUpperCase}}'s image gallery</h1>

    <ul>
      <li for-each="images">
        {{description}}:<br>
        <img src="{{src}}" alt="{{description}}"
           class="{{first $this ? 'first'}} {{last $this ? 'last'}}">
      </li>
    </ul>
  </div>
  <script src="bower_components/breezy/dist/breezy.js"></script>
  <script src="bower_components/observe-js/src/observe.js"></script>
  <script>
    var data = {
        site: { title: 'My page' },
        name: 'david',
        first: function(image) {
            return this.images.indexOf(image) === 0;
        },
        last: function(image) {
            return this.images.indexOf(image) === this.images.length - 1;
        },
        images: [{
            src: 'http://placehold.it/350x150',
            description: 'The first image'
        }, {
            src: 'http://placehold.it/350x150',
            description: 'Another image'
        }]
    };

    breezy.render(document.getElementById('application'), data);

    var counter = 0;
    // Lets make something happen
    setInterval(function() {
      data.images.push({
        src: 'http://placehold.it/350x150',
        description: 'Image #' + (counter++)
      });
    }, 2000);
  </script>
</body>
</html>
```

__Note:__ You can retrieve the context data from any DOM node using `breezy.context(node)`. With the above example:

```js
var node = document.getElementsByTagName('img')[0];
var image = breezy.context(node);

console.log(image);
// -> { src: 'http://placehold.it/350x150', description: 'The first image' }
```

If you didn't include ObserveJS you will have to call the renderer returned by `breezy.render` manually to update the view. If used properly this will probably be faster than observing objects. The end of the script then looks like:

```js
var renderer = breezy.render(document.getElementById('application'), data);

var counter = 0;
// Lets make something happen
setInterval(function() {
  data.images.push({
    src: 'http://placehold.it/350x150',
    description: 'Image #' + (counter++)
  });
  renderer();
}, 2000);
```

## Expressions

Breezy uses expressions as placeholders that will be substituted with the value when rendered. Expression are very similar to JavaScript property lookups, function calls and the tenary operator. A full expression looks like:

    path[.to.method] [args... ] [? truthy] [: falsy]

`path` is either a direct or dot-separated nested property lookup. `args` can be any number of (whitespace separated) parameters if the result of the path lookup is a function. Each parameter can either be another path or a sinlge- or doublequoted string. The optional truthy and falsy block can be used to change the return value to another value or string.

Examples:

- Look up the `name` property:
    - `name`
- Look up `site` and get the `title`:
    - `site.title`
- Get `name` and call the `toUpperCase` string method:
    - `name.toUpperCase`
- Call the `helpers.equal` method to check the name against a string:
    - `helpers.equal name 'David'`
- Call `helpers.equal` method and return `Yes` if it matches:
    - `helpers.equal name 'David' ? 'Yes'`
- Call `helpers.equal` method and return `No` if it does not match:
    - `helpers.equal name 'David' : 'No'`
- Call `helpers.equal` method and return `Yes` if and `No` if it does not match:
    - `helpers.equal name 'David' ? 'Yes' : 'No'`

`helpers.equal` simply looks like:

```js
{
  helpers: {
    equal: function(first, second) {
      return first === second;
    }
  }
}
```

Expressions can be used in [Attributes]() or any other text when wrapped with double curly braces `{{}}`:

```html
<div show-if="helpers.equal name 'David'">Hi {{name.toUpperCase}} how are you?</div>
<img src="person.png" alt="This person is: {{helpers.equal name 'David' ? 'Dave' : 'I don\'t know'}}">
```

__Note:__ Dynamically adding attributes like `<img {{helpers.equal name 'David' ? 'alt="This is David"'}}>` is currently not supported. This can almost always be done in a more HTML-y way, anyway, for example using a [custom attribute]().

## Context

Normally properties are looked up as you would expect, for example

```html
<img src="{{images.1.src}}" alt="{{images.1.description}}">
```

gets the attributes from the second image in the array. However, if the property is not found in the current context, Breezy will try to look it up at the parent and so on until we are at the root level (the data object you passed to the renderer). What this means is that for:

```html
<ul>
    <li for-each="images">{{site.title}}</li>
</ul>
```

The current context is the image we are currently iterating over, but, `site.title` is not a property of the current image. We will find it however one level higher at the root element.

There are also three *special properties* in any context:

- `$this` - Refers to the current context data (see the `{{first $this ? 'first'}}` example)
- `$key` - Is the property name the current context came from (e.g. the index of the image in the array)
- `$path` - The full path of the context. For example `images.0.src`

If you want to prevent lookups up the context, you can prefix the path with `$this` which will make something like

```html
<ul>
    <li for-each="images">{{$this.site.title}}</li>
</ul>
```

just output an empty string.

## Attributes

Breezy implements a small number of custom HTML5 attributes that can be used to show/hide elements, iterate over arrays or switch the context.

### for-each

Iterates over a list and renders the tag for each element.

```html
<ul>
  <li for-each="images">
    <img src="{{src}}" alt="{{description}}">
  </li>
</ul>
```

__Important:__ *Currently `for-each` only supports property lookups so you can not use the result of an expression.*

### show-if/show-if-not

Show the tag if an expression is truthy or falsy.

```html
<div show-if-not="images.length">No images</div>
<div show-if="images.length">There are {{images.length}} images.</div>
```

If `show-if` or `show-if-not` does not currently apply to the element, it will be replaced with an invisible element (`display: none;`) of the same type (we can't just skip it because missing elements will confuse the virtual-DOM). With `images.length === 0` the example would render like this:

```html
<div show-if-not="images.length">No images</div>
<div style="display: none;"></div>
```

### with

Switches the within this tag to the given property.

```html
<img with="images.1" src="{{src}}" alt="{{description}}">
```

__Important:__ *Currently `with` only supports property lookups so you can not use the result of an expression.*

## API

### context

In the Browser, `breezy.context(node)` returns the context data a DOM node has been rendered with. This is a great way to retrieve the data you want to modify.

```js
var node = document.getElementsByTagName('img')[0];
var image = breezy.context(node);

console.log(image);
// -> { src: 'http://placehold.it/350x150', description: 'The first image' }
image.src = 'http://placehold.it/350x150';
// -> view will update
```

### render

`breezy.render(content, data)` will render the given content. `content` can be an HTML template string and in the browser also a DOM Node which will then be replaced with the rendered content. `render` will return a string in NodeJS and in the Browser either a `DocumentFragment` (if `content` was a string) or a renderer function (if `content` was a DOM node).

### renderFile

`breezy.render(path, file, [callback])` renders a given file calling an optional callback. This is mainly for compatibility with [Express template engines](http://expressjs.com/guide/using-template-engines.html). If you want to create templates with an extension other than `.breezy` you can use this as the view engine:

```
var express = require('express');
var breezy = require('breezy');
var app = express();

app.engine('html', breezy.renderFile);
app.set('view engine', 'html');
app.set('views', __dirname + '/views');
```

### compile

`breezy.compile(content, options)` compiles a given template and returns a `renderer(data)` function. `content` can either be an HTML string or a DOM node. In Node, only strings are accepted and the renderer function will always return a string.

In the browser, if `content` is a string, a live-updating DocumentFragment will be returned the *first time* you call the renderer with data. Subsequent calls to that `renderer` are only possible with the same data or without any arguments and will update that DocumentFragment. If `content` is a DOM node the string representation of that node (`outerHTML`) will be used as the template and the node will be replaced with a live updating version.

```js
var renderer = breezy.compile('<div>Hello {{message}}</div>');

var data = { message: 'World' };
var result = renderer(data);
// `<div>Hello World</div> or DocumentFragment with div element

document.body.appendChild(result);

data.message = 'Welt';

// In the browser this will update the DOM
renderer();
// In Node, render it again
renderer(data);
```
## Examples

The [examples folder](https://github.com/daffl/breezy/tree/master/examples) contains the [Breezy TodoMVC implementation]() for both, the browser and Node with Express. To run them install Express and the TodoMVC common dependencies. In `/examples` run:

> npm install express
> cd todomvc
> bower install
> cd ..

You can run the Express application with

> node app.js

Then visit [http://localhost:3000/](http://localhost:3000/) to see the client side TodoMVC application with the full functionality.

At [http://localhost:3000/all](http://localhost:3000/all) the same template will be used but in Node generating some random Todos. Currently the server side example can only filter Todos ([/active](http://localhost:3000/active), [/complete](http://localhost:3000/complete)) but it should demonstrate how to use the shared data.

The application logic used on both sides is in `/public/js/view-model.js`. The file either exposes `window.ViewModel` on the Browser or exports the module for Node.