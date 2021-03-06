<h1 id="breezy">Breezy</h1>

Breezy is a view engine for JavaScript that renders a live-updating DOM (using [virtual-dom](https://github.com/Matt-Esch/virtual-dom/)) in the browser and strings in NodeJS. Create templates for client and server using HTML5 attributes and tags and simple but powerful [expressions](#breezy-expressions).

[![Travis build status](http://img.shields.io/travis/daffl/breezy/master.svg?style=flat)](https://travis-ci.org/daffl/breezy)

* [Breezy](#breezy)
    * [Getting started](#breezy-getting-started)
    * [Try it](#breezy-try-it)
    * [Usage](#breezy-usage)
        * [NodeJS](#breezy-usage-nodejs)
        * [Browser](#breezy-usage-browser)
    * [Expressions](#breezy-expressions)
    * [Context](#breezy-context)
    * [Attributes](#breezy-attributes)
        * [for-each](#breezy-attributes-for-each)
        * [show-if/show-if-not](#breezy-attributes-show-if-show-if-not)
        * [with](#breezy-attributes-with)
    * [API](#breezy-api)
        * [context](#breezy-api-context)
        * [render](#breezy-api-render)
        * [renderFile](#breezy-api-renderfile)
        * [compile](#breezy-api-compile)
    * [What's next?](#breezy-what-s-next-)
        * [TodoMVC example](#breezy-what-s-next--todomvc-example)
        * [Get involved](#breezy-what-s-next--get-involved)


<h2 id="breezy-getting-started">Getting started</h2>

Breezy uses custom HTML elements and attributes with [Expressions](#breezy-expressions) as placeholders to render HTML5 based templates. Lets create the following HTML template (e.g. in `page.html`):

```html
<!DOCTYPE html>
<html>
<head>
    <title>My image gallery</title>
</head>
<body>
  <div id="application">
    <h1>{{user.name.toUpperCase}}'s image gallery</h1>

    <ul>
      <li for-each="images">
          <img src="{{image}}" alt="{{title}}"
              class="{{isFirst $this ? 'first'}} {{isLast $this ? 'last'}}">
      </li>
    </ul>
  </div>
</body>
</html>
```

And use the following data for our image gallery:

```js
var data = {
  user: {
    username: 'daffl',
    name: 'David'
  },
  isFirst: function (image) {
    return this.images.indexOf(image) === 0;
  },
  isLast: function (image) {
    return this.images.indexOf(image) === this.images.length - 1;
  },
  images: [{
    "title": "First light",
    "link": "http://www.flickr.com/photos/37374750@N03/16032244980/",
    "image": "http://farm8.staticflickr.com/7525/16032244980_4052521ab6_m.jpg"
  }, {
    "title": "Yellow Daisy",
    "link": "http://www.flickr.com/photos/110649234@N07/16218828372/",
    "image": "http://farm8.staticflickr.com/7471/16218828372_5bc20dda73_m.jpg"
  }, {
    "title": "Striped Leaves",
    "link": "http://www.flickr.com/photos/110649234@N07/16033840027/",
    "image": "http://farm8.staticflickr.com/7538/16033840027_cd93d683e3_m.jpg"
  }]
};
```


Render it with Breezy in Node like

```js
var breezy = require('breezy');
var html = breezy.renderFile(__dirname + '/page.html', data);

console.log(html);
```

Or in the browser with:

```js
breezy.render(document.getElementById('application'), data);
```

The result will be:

```html
<!DOCTYPE html>
<html>
<head>
  <title>My image gallery</title>
</head>
<body>
  <div id="application">
    <h1>DAVID's image gallery</h1>

    <ul>
      <li for-each="images">
        <img src="http://farm8.staticflickr.com/7525/16032244980_4052521ab6_m.jpg"
          alt="First light" class="first">
      </li><li for-each="images">
        <img src="http://farm8.staticflickr.com/7471/16218828372_5bc20dda73_m.jpg"
          alt="Yellow Daisy" class="">
      </li><li for-each="images">
        <img src="http://farm8.staticflickr.com/7538/16033840027_cd93d683e3_m.jpg"
          alt="Striped Leaves" class="last">
      </li>
    </ul>
  </div>
</body>
</html>
```

<h2 id="breezy-try-it">Try it</h2>

The [following Fiddle](http://jsfiddle.net/Daff/z1b6of7k/light/) shows another image gallery example using jQuery to select an image. As your selection changes, the template will update automatically:

<iframe width="100%" height="380" src="http://jsfiddle.net/Daff/z1b6of7k/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


<h2 id="breezy-usage">Usage</h2>

Breezy can be used with NodeJS where it outputs a plain string or in the browser where a [virtual-dom](https://github.com/Matt-Esch/virtual-dom) is created which is then used to quickly update only the parts of the DOM that actually changed.

In the browser you will also get automatically updating templates as your data changes when [Polymer's ObserveJS](https://github.com/Polymer/observe-js) is included.

<h3 id="breezy-usage-nodejs">NodeJS</h3>

After

> npm install breezy

To use Breezy programmatically in Node just require it and call `.render(content, data)` or `.renderFile(path, data)`:

```js
var breezy = require('breezy');
var html = breezy.renderFile(__dirname + '/public/page.html', data);
console.log(html);

// Or compiled with a template string
var renderer = breezy.compile('<div>{{user.name}} (aka: {{user.username}})</div>');

console.log(renderer(data));
```

It can also be loaded as a view engine in your [Express](http://expressjs.com/) app:

```js
var express = require('express');
var app = express();

app.set('view engine', 'breezy');
app.set('views', __dirname + '/templates');
app.get('/', function(req, res) {
  res.render('page.html', data);
});

app.listen(3000);
```

<h3 id="breezy-usage-browser">Browser</h3>

The easiest way to get Breezy into the browser is via the [Bower](http://bower.io/) package:

> bower install breezy

You can also download the distributable from the [latest release](https://github.com/daffl/breezy/releases). Then include it in your page:

```html
<script src="bower_components/breezy/dist/breezy.js"></script>
```

`dist/breezy.js` can also be loaded as an AMD and CommonJS module. If included without a module loader, the global variable `breezy` will be available.

Breezy has no hard dependencies but if you want your templates to automatically update when the displayed data change, you will also need to install [ObserveJS](https://github.com/Polymer/observe-js):

> bower install observe-js

And include it in the page as well:

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
  <h1>{{user.name.toUpperCase}}'s image gallery</h1>

  <ul>
    <li for-each="images">
      <img src="{{image}}" alt="{{title}}"
           class="{{isFirst $this ? 'first'}} {{isLast $this ? 'last'}}">
    </li>
  </ul>
</div>
<script show-if-not="isNode" src="../../dist/breezy.js"></script>
<script show-if-not="isNode" src="bower_components/observe-js/src/observe.js"></script>
<script show-if-not="isNode">
  var data = {
    user: {
      username: 'daffl',
      name: 'David'
    },
    isFirst: function (image) {
      return this.images.indexOf(image) === 0;
    },
    isLast: function (image) {
      return this.images.indexOf(image) === this.images.length - 1;
    },
    images: [{
      "title": "First light",
      "link": "http://www.flickr.com/photos/37374750@N03/16032244980/",
      "image": "http://farm8.staticflickr.com/7525/16032244980_4052521ab6_m.jpg"
    }, {
      "title": "Yellow Daisy",
      "link": "http://www.flickr.com/photos/110649234@N07/16218828372/",
      "image": "http://farm8.staticflickr.com/7471/16218828372_5bc20dda73_m.jpg"
    }, {
      "title": "Striped Leaves",
      "link": "http://www.flickr.com/photos/110649234@N07/16033840027/",
      "image": "http://farm8.staticflickr.com/7538/16033840027_cd93d683e3_m.jpg"
    }]
  };

  breezy.render(document.getElementById('application'), data);

  var counter = 0;
  // Lets make something happen
  setInterval(function () {
    data.images.push({
      image: 'http://placehold.it/240x160',
      title: 'Placeholder #' + (counter++),
      link: '#'
    });
  }, 2000);
</script>
</body>
</html>
```

If you don't include ObserveJS you will have to call the renderer returned by `breezy.render` manually to update the view. If used properly this will probably be faster than observing objects. The end of the script then looks like:

```js
var renderer = breezy.render(document.getElementById('application'), data);

var counter = 0;
// Lets make something happen
setInterval(function () {
  data.images.push({
    image: 'http://placehold.it/240x160',
    title: 'Placeholder #' + (counter++),
    link: '#'
  });
  renderer();
}, 2000);
```

__Note:__ You can retrieve the context data from any DOM node using `breezy.context(node)`. With the above example:

```js
var node = document.getElementsByTagName('img')[0];
var image = breezy.context(node);

console.log(image);
```

This makes it easy to get and modify the data in event listeners etc.

<h2 id="breezy-expressions">Expressions</h2>

Breezy uses expressions as placeholders that will be substituted with the value when rendered. Expression are very similar to JavaScript property lookups and function calls with the tenary operator. A full expression looks like:

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
- Call `helpers.equal` method and return `Yes` if it matches (`null` otherwise):
    - `helpers.equal name 'David' ? 'Yes'`
- Call `helpers.equal` method and return `No` if it does not match (`null` otherwise):
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

Expressions can be used in [Attributes](#breezy-attributes) or any other text when wrapped with double curly braces `{{}}`:

```html
<div show-if="helpers.equal name 'David'">Hi {{name.toUpperCase}} how are you?</div>
<img src="person.png" alt="This person is: {{helpers.equal name 'David' ? 'Dave' : 'I don\'t know'}}">
```

__Note:__ Dynamically adding attributes like `<img {{helpers.equal name 'David' ? 'alt="This is David"'}}>` is currently not supported. This can almost always be done in a more HTML-y way, anyway, for example using a *custom attribute*.

<h2 id="breezy-context">Context</h2>

Normally properties are looked up as you would expect, for example

```html
<img src="{{images.1.src}}" alt="{{images.1.description}}">
```

gets the attributes from the second image in the array. However, if the property is not found in the current context, Breezy will try to look it up at the parent and so on until we are at the root level (the data object you passed to the renderer). What this means is that for

```html
<ul>
    <li for-each="images">{{site.title}}</li>
</ul>
```

where the current context is the image we are currently iterating over, `site.title` is not a property of the current image. We will find it however one level higher at the root element.

There are also three *special properties* in any context:

- `$this` - Refers to the current context data (see the `{{first $this ? 'first'}}` example)
- `$key` - Is the property name the current context came from (e.g. the index of the image in the array)
- `$path` - The full path of the context. For example `images.0.src`

If you want to prevent lookups up the context you can prefix the path with `$this` which will make something like

```html
<ul>
    <li for-each="images">{{$this.site.title}}</li>
</ul>
```

just output an empty string.

<h2 id="breezy-attributes">Attributes</h2>

Breezy implements a small number of custom HTML5 attributes that can be used to show/hide elements, iterate over arrays or switch the context.

<h3 id="breezy-attributes-for-each">for-each</h3>

Iterates over a list and renders the tag for each element.

```html
<ul>
  <li for-each="images">
    <img src="{{src}}" alt="{{description}}">
  </li>
</ul>
```

__Important:__ *Currently `for-each` only supports property lookups so you can not use the result of an expression.*

<h3 id="breezy-attributes-show-if-show-if-not">show-if/show-if-not</h3>

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

<h3 id="breezy-attributes-with">with</h3>

Sets the context for this tag to the given data:

```html
<img with="images.1" src="{{src}}" alt="{{description}}">
```

__Important:__ *Currently `with` only supports property lookups so you can not use the result of an expression.*

<h2 id="breezy-api">API</h2>

<h3 id="breezy-api-context">context</h3>

In the Browser, `breezy.context(node)` returns the context data a DOM node has been rendered with. This is a great way to retrieve the data you want to modify.

```js
var node = document.getElementsByTagName('img')[0];
var image = breezy.context(node);

console.log(image);
// -> { src: 'http://placehold.it/350x150', description: 'The first image' }
image.src = 'http://placehold.it/350x150';
// -> view will update
```

<h3 id="breezy-api-render">render</h3>

`breezy.render(content, data)` will render the given content. `content` can be an HTML template string and in the browser also a DOM Node which will then be replaced with the rendered content. `render` will return a string in NodeJS and in the Browser either a `DocumentFragment` (if `content` was a string) or a renderer function (if `content` was a DOM node).

<h3 id="breezy-api-renderfile">renderFile</h3>

`breezy.render(path, file, [callback])` renders a given file calling an optional callback. This is mainly for compatibility with [Express template engines](http://expressjs.com/guide/using-template-engines.html). If you want to create templates with an extension other than `.breezy` you can use this as the view engine:

```js
var express = require('express');
var breezy = require('breezy');
var app = express();

app.engine('html', breezy.renderFile);
app.set('view engine', 'html');
app.set('views', __dirname + '/views');
```

<h3 id="breezy-api-compile">compile</h3>

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
<h2 id="breezy-what-s-next-">What's next?</h2>

<h3 id="breezy-what-s-next--todomvc-example">TodoMVC example</h3>

The [examples folder](https://github.com/daffl/breezy/tree/master/examples) contains the [Breezy TodoMVC implementation](http://daffl.github.io/breezy/todomvc/) for both, the browser and Node with Express. The template is located in [examples/public/index.html](https://github.com/daffl/breezy/blob/master/examples/public/index.html). To run them install Express and the TodoMVC common dependencies. In `/examples` run:

> npm install express && cd todomvc && bower install && cd ..

You can run the Express application with

> node app.js

Then visit [http://localhost:3000/](http://localhost:3000/) to see the client side TodoMVC application with the full functionality.

At [http://localhost:3000/all](http://localhost:3000/all) the same template will be used but in Node generating some random Todos. Currently the server side example can only filter Todos ([/active](http://localhost:3000/active), [/complete](http://localhost:3000/complete)) but it should demonstrate how to use the shared data model.

The application logic used on both sides is in `/public/js/view-model.js`. The file either exposes `window.ViewModel` on the Browser or exports the module for Node.

<h3 id="breezy-what-s-next--get-involved">Get involved</h3>

Breezy is still very new and there will be issues and many features to come. If you want to help or have any questions or comments, just open a [GitHub issue](https://github.com/daffl/breezy/issues) or ping [@daffl](https://twitter.com/daffl) on Twitter.

