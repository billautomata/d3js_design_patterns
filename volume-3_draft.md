# What can you do with a selection?

A selection is a javascript object with a collection of useful methods.

It is your starting point for creating and removing DOM [elements](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement). You also use selections to update how those elements appear on the page and respond to events.

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


## append & insert
Calling `append` creates a DOM element as a child of the parent selection.
```javascript
// before
// <body> </body>

var parent_selection = d3.select('body')
var child_selection = parent_selection.append('div')

// after
// <body> <div></div> </body>
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
When you call `append` it returns a javascript object.  That object is a new selection that represents the element(s) created.

`insert` works just like append, but there is a second argument which is a selector string you would pass to `d3.select`.

```javascript
var body = d3.select('body')
var div_one = body.append('div').attr('id','baz').html('foo')
var div_two = body.insert('div', 'div#baz').html('bar')

// <body>
//   <div> bar </div>           // div two
//   <div id='baz'> foo </div>  // div one
// </body>
```




## attr [`mdn reference`](https://developer.mozilla.org/en-US/docs/Web/API/Element/attributes)

```javascript
div_selection.attr('foo', 'bar')
```

The above code results in a `<div>` element with the attribute `foo`.  

```javascript
<div foo='bar'></div>
```

To read the value of the attribute, call the `attr` function with only one argument (the attribute you want to read the value of).

```javascript
var m = div_selection.attr('foo')
console.log(m) // 'bar'
```

This pattern is all over the place in d3.  If you are setting something with two arguments, you are commonly able to read that value back by only passing that first argument to the same function.

[Common](https://css-tricks.com/the-difference-between-id-and-class/) attributes are `class` and `id`.

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

##### book-keeping
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

## text & html
Most d3 code is meant to act on the attributes of elements.  `text` and `html` let you change the content of the element.
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

## on
Events!

```javascript
d3.select('body').on('click', function(){

})
```


## node
## data & datum
## each









```javascript
// [ "classed", "style", "property", "insert", "remove", "data", "datum", "filter", "order",
//   "sort", "each", "call", "empty", "node", "size", "on", "transition",
//   "interrupt"]
```
