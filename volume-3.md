# What can you do with a selection?

D3 returns a javascript object when you call `d3.append()` `d3.select()` and `d3.selectAll()`.  This javascript object is referred to as a selection.

The selection is your entry point for finding, creating, and removing DOM [elements](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement). You also use selections to apply styles and attributes that dictate how those elements appear on the page and respond to events.

```javascript
// create a selection by calling d3.select('tagname')
var body_selection = d3.select('body')

// create a selection by calling .append('tagname') on another selection
var div_selection = body_selection.append('div')

// list the methods available to all d3 selections
Object.keys(div_selection.__proto__)

// ["select", "selectAll", "attr", "classed", "style", "property", "text",
//   "html", "append", "insert", "remove", "data", "datum", "filter", "order",
//   "sort", "each", "call", "empty", "node", "size", "on", "transition",
//   "interrupt"]

```

## `append()` & `insert()`
Calling `append()` creates a DOM element as a child of the parent selection for each member in the parent selection.
```javascript
// before
// <body> </body>

var parent_selection = d3.select('body')
var child_selection = parent_selection.append('div')

// after
// <body>   <div></div>     </body>
// <parent> <child></child> </parent>
```
```javascript
// before
// <div> </div>
// <div> </div>

var parent_selection = d3.selectAll('div')
var child_selection = parent_selection.append('p')

// after
// <div> <p></p> </div>
// <div> <p></p> </div>

child_selection.size() // 2

```
When you call `append()` it returns a javascript object.  That object is a new selection that represents the element(s) created.

`insert()` works just like append, but there is a second argument which is a selector string you would pass to `d3.select()`.

```javascript
var body = d3.select('body')
var div_one = body.append('div').attr('id','baz').html('foo')
var div_two = body.insert('div', 'div#baz').html('bar')

// <body>
//   <div> bar </div>           // div two
//   <div id='baz'> foo </div>  // div one
// </body>
```

## `attr()` [`mdn reference`](https://developer.mozilla.org/en-US/docs/Web/API/Element/attributes)

```javascript
div_selection.attr('foo', 'bar')
```

The above code results in a `<div>` element with the attribute `foo`.  

```javascript
<div foo='bar'></div>
```

To read the value of the attribute, call the `attr` function with only one argument, the attribute you want to read the value of.

```javascript
var m = div_selection.attr('foo')
console.log(m) // 'bar'
```

This pattern is all over the place in d3.  If you are setting something with two arguments, you are commonly able to read that value back by only passing that first argument to the same function.

Aside from the mandatory attributes required to render certain SVG elements (like `x` and `width`) the most [common](https://css-tricks.com/the-difference-between-id-and-class/) attributes are `class` and `id`.  

```javascript
selection.append('div').attr('id', 'foo')
// <div id='foo'></div>
```

#### what else can you do with this?
##### search by attribute name and value
```javascript
var k = d3.selectAll('[foo=bar]')
```
Calling `selectAll` this way returns all the DOM elements where the attribute key/value pattern matches the pattern specified in the argument selector string `'[foo=bar]'`.

##### state tracking
You can keep track of the state for an element by reading and writing the value of an attribute for that element during a mouse event.
```javascript
// set the attribute
div_selection.attr('my_clicked', 'false')

// setup an event for the dom element
div_selection.on('click', function(){

    // read the attribute value
    var clicked = d3.select(this).attr('my_clicked')

    if(clicked === 'false'){
      alert('clicked for the first time!')

      // write the attribute value
      d3.select(this).attr('my_clicked', 'true')

    } else {
      alert('clicked NOT for the first time')
    }
})
```

##### external library interoperability
Frameworks like [AngularJS](https://docs.angularjs.org/guide/directive) can rely on the presence of attributes to determine the behavior of the app.  Setting these attributes and values on the DOM elements is how you can hook in these external libraries

## `text()` & `html()`
Most d3 code is meant to act on the attributes of elements.  `text()` and `html()` let you change the content of the element.
```javascript
// before
// <div id='foo'> </div>

d3.select('div#foo').text('bar')

// after
// <div id='foo'> bar </div>
```
```javascript
// before
// <div id='foo'> </div>
// <div id='bar'> </div>

d3.select('div#foo').html('<p>bar</p>')
d3.select('div#bar').append('p').text('baz')

// after
// <div id='foo'> <p> bar </p> </div>
// <div id='bar'> <p> baz </p> </div>
```

## `on()` & `d3.event`
The `on()` function allows you to attach a callback when an event is triggered.

```javascript
var selection = d3.select('body')

// selection.on('eventType', callbackFunction)

selection.on('click', function(){
  console.log('an event was triggered')
})
```

The [list](https://developer.mozilla.org/en-US/docs/Web/Events) of possible DOM events is very long.  The most common events are `mouseover` `mouseout` `click` `dblclick` `keyup` `keydown` `load` and `resize`.

If you attach your events with the `d3.on` function you will have access to a global `d3.event` object with meta-data about your event inside your callback.

```javascript
var selection = d3.select('body')
selection.on('click', function(){
  console.log(Object.keys(d3.event.__proto__))
})

// ["screenX", "screenY", "clientX", "clientY", "ctrlKey", "shiftKey", "altKey",
// "metaKey", "button", "buttons", "relatedTarget", "pageX", "pageY", "x", "y",
// "offsetX", "offsetY", "movementX", "movementY", "fromElement", "toElement",
//  "which", "webkitMovementX", "webkitMovementY", "layerX", "layerY",
//  "initMouseEvent"]  
```

```javascript
// with d3.select(this)
var selection = d3.select('body')

function bg_red(){
  d3.select(this).style('background-color', 'red')
}
function bg_blue(){
  d3.select(this).style('background-color', 'blue')
}
selection.on('mouseover', bg_red)
selection.on('mouseout', bg_blue)
```
Calling `d3.select(this)` from inside an event callback will select the original parent element so you don't have to keep track of which selection is calling the function.  This is very useful if you want to apply the same function in response to many different types of events.  Without `d3.select(this)` the above code would look like this.
```javascript
// without d3.select(this)
function bg_red(sel){
  sel.style('background-color', 'red')
}
function bg_blue(sel){
  sel.style('background-color', 'blue')
}

var selection = d3.select('body')

selection.on('mouseover', function(){
  bg_red(selection)
})
selection.on('mouseout', function(){
  bg_blue(selection)
})
```
While it's not a huge deal to have to wrap your event functions within other anonymous functions, it is definitely helpful when you don't need to.

## `data()` & `datum()`
The standard `select().data().enter()` [pattern](http://bost.ocks.org/mike/circles/) is well documented territory.  Manual use of `data()` and `datum()` is documented in this repo [volume 2 • d3js without enter](volume-2.md).

## `node()`
The `node()` function provides access to the lowest level [DOM operations](https://developer.mozilla.org/en-US/docs/Web/API/Node) attached to the DOM Node.

```javascript
var selection = d3.select('body').append('div').html('zomg')

selection.attr('foo', 'bar')

// <body> <div foo='bar'> zomg </div> </body>

var n = selection.node()

// common useful things found on the node

console.log(n.nodeName)         // 'div'
console.log(n.attributes)       // { 0: 'foo' }
console.log(n.getClientRects()) // returns the bounding box for the element
console.log(n.hidden)           // false
console.log(n.innerHTML)        // 'zomg'
console.log(n.hasChildNodes())  // false

// append a child node
selection.append('div').html('a child node')
console.log(n.hasChildNodes())  // true

```
