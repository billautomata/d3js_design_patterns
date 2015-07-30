#d3js and the first-class function

Passing a function as an argument to other functions is a difficult concept to understand for beginners, especially those coming from other languages.  D3 further compounds this fact by making this model integral to the basic pattern, but doesn't force the designer to understand what the heck is going on when they type `function` in the middle of their code.  

Instead of learning what the underlying code is doing, they just blindly type...

```javascript
  // ...
  .append('rect')
  .attr('fill', function(d,i){ return 'brown' })
  .attr('stroke', function(d,i){ return 'none' })
```

...even when it clearly isn't necessary.  I swear code that looks like this is written every day by beginner and intermediate d3 designers because, while it doesn't explicitly blow up their code, it still works.  It runs.

This article will try to make sure it doesn't happen again.  We will cover what d3 is doing when you pass a function instead of a value.  By the end you will have a better understanding of a wonderfully useful part of javascript.

### function(d,i){ return d; }

Almost all D3 code takes this form:

```javascript
var p = [ 3, 1, 4, 1, 5]
var svg = d3.select('body').append('svg')
svg.selectAll('g.foo')
  .data(p)
  .enter()
    .append('g').attr('class','foo')
      .append('rect')
      .attr('id', 'zomg')
      .attr('x', function(d,i){
        return (d*10.0) // what the heck is this?
      })
      // ...
```

In the above code we create an `svg` element.  Then append a series of `g` (group) elements to the `svg`. Then append a `rect` element to that group element.  

When creating that `rect` element I set the `id` [attribute](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute) to a string `'zomg'`.  `'zomg'` is a value, like `0` or `true`.  That value is passed as the second argument in the `attr()` call.

In the next line of code where we call `attr` and set the `x` attribute, instead of passing in a value like the number `108.1`, I pass a function as an argument.  
### usually

Usually you write a function ahead of time, then call it later when you need it, right?  You may not have ever passed a function as an argument before.  

```javascript
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
// this is a little different
function add(a,b){
  return a() + b()
}
//               first argument         second argument
var a = add(function(){ return 3 }, function(){ return 1 })
// a = 4
```

The above code is weird.  Why would you ever do this?  There really is no purpose to passing in those functions as arguments.  The functions I am passing always return a single value.  Everything but the `3`, in that first argument is superfluous.

The ability to pass a function as an argument becomes important when you do more than just `return 3` in the function you are passing.  What if the "argument function" has complicated behavior like making a call to a server to checks to see if the user is authenticated?  

Being able to pass a function as an argument to other functions lets you more easily separate concerns.  It should help you construct your programs in a more maintainable fashion.  

You are also probably doing it already without recognizing it.

```javascript
// common jquery event code
$('#foo').click(function(){
  $('#bar').hide()
})
```
The above code attaches an event handler to DOM elements.  A function is passed as an argument to the `click` function, that "argument function" will be run whenever a click event occurs on a DOM element with the id of `foo`.

```javascript
// this is a lot different
function adder_maker(a){
  var c = a
  return function d(b){
    return c() + v
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

### functions are objects

You can declare a function anywhere you could put a value.

```javascript
function foo(zomg){

  bar(zomg)

  // another function declaration inside an existing function
  function bar(a){
    console.log(a + ' hahah ' + a)
  }

}

function baz(lol){
  console.log(lol + ' hehehehe ' + lol)
  // you cannot run bar() in this function because baz has no
  // bar() to call, foo owns bar()
}

```

In the first example
```javascript
// ...
.attr('x', function(d,i){
  return (d*10.0) // what the heck is this?
})
// ...
```

Why did we pass a function to the second argument of `attr`?  That function you pass as an argument is saved and attached to the `rect` element. (see [related d3 source code](https://github.com/mbostock/d3/blob/master/src/selection/attr.js#L58)).

Why is the function saved and attached to the `rect` element?  The function is run for each element in the array you passed to `data()`.  It generally evaluates

In the example above

```javascript
var p = [ 3, 1, 4, 1, 5]
var svg = d3.select('body').append('svg')
svg.selectAll('g.foo')
  .data(p)
  .enter()
    .append('g').attr('class','foo')
      .append('rect')...
      ...
```

Because you have 5 elements in your `p` array, you will append 5 `g` elements with the class `foo`.  One for each element in the `p` array.  Each of those `g` elements has a `rect` element appended.

For every `rect` element that is appended to the svg, the anonymous in-line callback function you passed as the second argument, is run when the `x` attribute is set for that `rect`.  Each time that function is run, the elements in the `p` array are passed as arguments to the function.

```javascript
// example
function(3,0){  // run the function, pass in p[0] and 0
  return (3*10) // 30
}
function(1,1){  // run the function, pass in p[1] and 1
  return (1*10) // 10
}
function(4,2){  // run the function, pass in p[2] and 2
  return (4*10) // 40
}
function(1,3){  // run the function, pass in p[3] and 3
  return (3*10) // 30
}
function(5,4){  // run the function, pass in p[4] and 4
  return (5*10) // 50
}
```

* * *







The usual chain of functions for a d3js visualization can seem to come to an abrupt end with the introduction of the word `function`.  For some students their brain shuts down
