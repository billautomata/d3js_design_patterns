# d3 without .enter()

The first [d3 tutorial](http://bost.ocks.org/mike/bar/) people read will have them passing an array to `.data()` after `selectAll`ing a bunch of DOM elements that don't exist yet.  

> Since we know the selection is empty, the returned update and exit selections are also empty, and we need only handle the enter selection which represents new data for which there was no existing element. We instantiate these missing elements by appending to the enter selection.

`from the official page`

[Articles](http://alignedleft.com/tutorials/d3/binding-data) have been written about how this all works under the hood.

The concepts are difficult for a beginner to comprehend. I would also argue the pattern is a hinderance in some situations.

[This](https://www.dashingd3js.com/binding-data-to-dom-elements) is a great article that explains the process as "joining your data array with a virtual selector that has enter, update, and exit methods."  A huge benefit of this model is that you can automatically update, and remove, elements by passing a new dataset to the selection.

If you find this model a roadblock to getting started, or if you just prefer not to use it in your d3 programs, you can easily ignore it.

###[example](http://codepen.io/billautomata/pen/zGapOR/)

```javascript
// our data array
var data = [3, 4, 1, 9, 2]

// boilerplate setup code for our containers and scales
var body = d3.select('body')

var color_scale = d3.scale.linear()
  .domain([0, d3.max(data)])
  .range([100, 255])

// parent containers for our example
var example1 = body.append('div').attr('id', 'container')
var example2 = body.append('div').attr('id', 'container')

// creating the elements using .data().enter()
example1.selectAll('div.foo')
  .data(data)
  .enter()
  .append('div')
  .attr('class', 'foo')
  .style('background-color', 'white')
  .html(String)

// performing the same transition on all elements
d3.selectAll('div.foo')
  .transition()
  .duration(2000)
  .style('background-color', function(d) {
    var v = color_scale(d)
    return d3.rgb(v, v, v)
  })

// creating the elements without using .enter()
var divs = [] // this is where we will save our elements

// iterate over each element in the array
data.forEach(function(d) {

  var div = example2.append('div')
    .datum(d)
    .html(d)
    .style('background-color', 'white')

  // push the created object to an array
  divs.push(div)

})

// perform the same transition on each element
divs.forEach(function(div) {

  div.transition()
    .duration(2000)
    .style('background-color', function(d) {
      var v = color_scale(d)
      return d3.rgb(v, v, v)
    })

})
```

### why do this?

I prefer when I am done setting up my graph, to have an iterable array of javascript objects that I can `transition` or `remove` individually.  I have found that cache-ing individual element selectors on an array is a workable model for scaling apps that don't rely on querying the DOM by element type, class or ID.

There are many reasons to use the `.data().enter()` pattern, but if you do not find it convenient you can do things manually.
