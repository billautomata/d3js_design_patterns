# d3js and the first-class function

Passing a function as an argument to other functions is a difficult concept to understand for beginners, especially those coming from other languages.  D3 further compounds this difficulty by making the functions-as-arguments model integral to the basic pattern, but doesn't force the designer to understand what the heck is going on when they type `function` in the middle of their code.  

Instead of learning what the underlying code is doing, some just blindly type...

```javascript
// figure 4.1
  // ...
  .append('rect')
  .attr('fill', function(d,i){ return 'brown' })
  .attr('stroke', function(d,i){ return 'none' })
```

...even when it clearly isn't necessary.  I swear code that looks like this is written every day by beginner and intermediate d3 designers because, while it doesn't explicitly blow up their code, it still works.  It runs.

This article will try to make sure it doesn't happen again.  We will cover what d3 is doing when you pass a function instead of a value.  By the end you will have a better understanding of a wonderfully useful part of javascript.

### `function(d,i){ return d; }`

Almost all D3 code takes this form:

[codepen link](http://codepen.io/billautomata/pen/xGmKEW/)

```javascript
// figure 4.2
var p = [ 3, 1, 4, 1, 5]

var svg = d3.select('body').append('svg')

var elements = svg.selectAll('rect#foo')
  .data(p)
  .enter()
    .append('rect')
    .attr('id', 'zomg')
    .attr('x', function(d){
      return (d*10.0) // what the heck is this?
    })
    .attr('y',0).attr('width',4).attr('height',200)
```

In the above code we create an `svg` element.  Then append a series of `rect` elements to the `svg`.

When creating that `rect` element we set the `id` [attribute](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute) to a string, `'zomg'`.  `'zomg'` is a value, like `0` or `true`.  That value is passed as the second argument in the `attr()` call.

### `attr(name, value)`

In the next line of code where we call `attr` and set the `x` attribute, instead of passing in a value like the number `108.1`, I pass a function as an argument.  
### `attr(name, function(){ return value })`

### usually

```javascript
// figure 4.3
// this is normal-ish looking code
function add(a,b){
  return (a+b)
}
var a = add(2,2)
// a = 4 (2 + 2)

var b = add(8,a)
// b = 12 (4 + 8)

```

I write a function `add`.  Then I use it to add stuff together.

```javascript
// figure 4.4
// this is a little different
function add(a,b){
  return a() + b()
}
//               first argument         second argument
var a = add(function(){ return 3 }, function(){ return 1 })
// a = 4
```

The above code is weird.  Why would you ever do this?  

There really is no purpose to passing in those functions as arguments, I am just doing it to show that you can.  The functions I am passing always return the same value.  Everything but the `3`, in that first argument is superfluous.

The ability to pass a function as an argument becomes important when you do more than just `return 3` in the function you are passing.  What if the "argument function" has complicated behavior like making a call to a server to check to see if the user is authenticated?  

Being able to pass a function as an argument to other functions lets you more easily separate concerns.  It should help you construct your programs in a more maintainable fashion.  

You are also probably doing it already without recognizing it.

```javascript
// figure 4.5
// common jquery event code
$('#foo').click(function(){
  $('#bar').hide()
})
```
The above code attaches an event handler to DOM elements.  A function is passed as an argument to the `click` function, that "argument function" will be run whenever a click event occurs on a DOM element with the id of `foo`.

```javascript
// figure 4.6
// this is a lot different
function adder_maker(a){
  var c = a
  return function d(b){
    return c() + b
  }
}

var random_adder = adder_maker(
    function(){ return Math.random() }  // a, in adder_maker
  )

random_adder(1) // 1.1
random_adder(1) // 1.93
random_adder(1) // 1.03

var static_adder = adder_maker (
    function(){ return 4 }
  )

static_adder(1) // 5
static_adder(3) // 7
```

The above code is even weirder.  The function you pass as an argument `a` is assigned the variable `c`.  Then yet another function `d` is returned that will evaluate `c` and add what is returned from `c` to `b`, and return all of that finally.

Here we assign the returned function `d` to the var `random_adder`.

Now when you call `random_adder` many times you should get a new number each time.  This is because `Math.random()` is newly evaluated with each call of `random_adder` because a *reference* to the function `a` is stored as `c` and returned in the function `d` that we assign to the variable `random_adder`.

### back to d3

In the first example...
```javascript
// ...
.attr('x', function(d,i){
  return (d*10.0) // what the heck is this?
})
// ...
```

Why are we passing a function to the second argument of `attr`?  That function you pass as an argument is saved and attached to the `rect` element. (see [related d3 source code](https://github.com/mbostock/d3/blob/master/src/selection/attr.js#L58)).

Why is the function saved and attached to the `rect` element?  The function is run for each element in the array you passed to `data()`.  

```javascript
// figure 4.2 (again)
var p = [ 3, 1, 4, 1, 5]

var svg = d3.select('body').append('svg')

var elements = svg.selectAll('rect#foo')
  .data(p)
  .enter()
    .append('rect')
    .attr('id', 'zomg')
    .attr('x', function(d){
      return (d*10.0) // what the heck is this?
    })
    .attr('y',0).attr('width',4).attr('height',200)
```

Because you have 5 elements in your `p` array, you will append 5 `rect` elements with the id `foo`.  One for each element in the `p` array.

For every `rect` element that is appended to the svg the anonymous in-line callback function, you passed as the second argument to the `attr` function, is run when the `x` attribute is being set for each `rect`.  Each time that function is run, the elements in the `p` array are passed as arguments to the anonymous function, and whatever that function returns determines the `x` attribute value.

```javascript
// figure 4.7
// example
// first rect is created
function(3){  // run the function, pass in p[0]
  return (3*10) // 30
}

// second rect is created
function(1){  // run the function, pass in p[1]
  return (1*10) // 10
}

// third rect is created
function(4){  // run the function, pass in p[2]
  return (4*10) // 40
}

// fourth rect is created
function(1){  // run the function, pass in p[3]
  return (1*10) // 10
}

// fifth rect is created
function(5){  // run the function, pass in p[4]
  return (5*10) // 50
}
```

The results of running the code from `Figure 4.2` are below.

```javascript
// figure 4.8
<svg>
  <rect id='zomg' x=30 y=0 width=4 height=200></rect>
  <rect id='zomg' x=10 y=0 width=4 height=200></rect>
  <rect id='zomg' x=40 y=0 width=4 height=200></rect>
  <rect id='zomg' x=10 y=0 width=4 height=200></rect>
  <rect id='zomg' x=50 y=0 width=4 height=200></rect>
</svg>
```

### why go through all the trouble of this?

#### easier transitions

If you follow the pattern during an `update > transition` cycle, you can just call `data` again, pass in a new array, call `transition`, then list attributes.

```javascript
// figure 4.9
// update the bound data
elements.data([ 1, 2, 3, 6, 8])

elements.transition()
  .delay(1000)
  .duration(3000)
  .attr('x', function(d) { return (d*10.0) })
```

Now all the elements will have updated `x` attributes and transition to new positions on the screen.  The code `function(d) { return (d*10.0) }` will be evaluated again for each element in the `data` array.

```javascript
// figure 4.10
// new x attributes from new data array [1,2,3,6,8]
<svg>  //           \/ here
  <rect id='zomg' x=10 y=0 width=4 height=200></rect>
  <rect id='zomg' x=20 y=0 width=4 height=200></rect>
  <rect id='zomg' x=30 y=0 width=4 height=200></rect>
  <rect id='zomg' x=60 y=0 width=4 height=200></rect>
  <rect id='zomg' x=80 y=0 width=4 height=200></rect>
</svg>
```

#### better understanding of javascript in general

If you already understood functions as arguments, assigning functions to values, and the overall idea of a [function as a first-class citizen](https://en.wikipedia.org/wiki/First-class_function), then I'm sorry if I talked down to you during this article.

Most d3 programmers begin in a world where this is not a common programming pattern.  Callbacks are usually the domain of network operations and other I/O. If you fully understood the [javascript callback pattern](http://callbackhell.com/) before starting with D3, then you had one less roadblock on the seemingly steep D3 learning curve.

### copy-pasta

`window.onload = function(){ ...` is easy to write, and it works even when you don't know what it is doing. You don't have to know that you are assigning an anonymous callback function to the window event handler.  Your code in between the `{}` brackets will work just fine.

On the other hand, you can't use D3 effectively if you don't know when to pass a callback, and when to just pass a value.  D3 will also be a minefield of frustration if you use [data-binding](http://alignedleft.com/tutorials/d3/binding-data) but don't understand that the data you are binding is later used as the argument of an callback function you also write.
