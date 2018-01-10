# Understanding some Javascript

So I've been working on a Javascript project recently, and I am by no means a front end developer. But occasionally I've called myself a full stack developer, so currently I'm the only one in the office with Javascript experience so I got to do most of the heavy lifting. This is also the same project in which I was doing the ops work, since again, I'm "full stack". 

For the most part, I'm knowledgeable enough to be dangerous within Javascript, but the issue is that I haven't spent enough time working within larger projects, so I lack the intuition in knowing how to structure a larger Javascript project. What I'm good at is being able to have 1 or 2 `.js` files that will then contain functions that do my bidding, usually make a chart. 

## The task
Something I needed to get done within this project was to hit an api endpoint, fetch the data, and then display it in a table. I figured this would be fairly easily done in Javascript using iteration. 

1. Hit api endpoint, get data object
2. Iterate through data object
3. Create row/cells, and attach event handler to values in first column

I didn't think too much about this, since it appeared to be fairly straightforward from the start. I wanted the first column to be clickable, and it would use the value in that cell within a function and do something.

## The code
This could probably be a test question, figure out what happens when the `onclick` event handler is triggered: 


```
for (var key in data){
  var link = document.createElement('a');
  var newRow   = tableRef.insertRow(-1);
  var newCell  = newRow.insertCell(0);
  var cellValue  = document.createTextNode(key);
  link.appendChild(cellValue);
  link.onclick = function () {alert(key);};
  newCell.appendChild(link);

  var addyCell = newRow.insertCell(1);
  var addy = data[key].address;
  var addyNode = document.createTextNode(addy);

  addyCell.appendChild(ad
```

This is how I first wrote the code, and then tested it. You can see a (working jsfiddle)[https://jsfiddle.net/3xb70dn8/] here.

## What I forgot
So when the code runs, it creates rows/cells in a predefined table that is referenced by `tableRef`. 1 row with 2 cells per key/value in the data object. But what I didn't first understand is why the `onclick` handler always gave the value of the last key. So instead of say `key1` being alerted when you clicked on the key1 cell, it would give you `key3`. This seemed to me that the `onclick` handler gets assigned after the Javascript code is executed, and then all handlers get the last value.

# Scope and hoisting
This is something that I totally forgot about Javascript. Hoisting tells you when a variable is able to be accessed, so you can have variables that are accessed globally, or locally. The most common statement is the (`var` statement)[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var] that is used to define a variable that is hoisted to the top of the function scope. This means that in my code where I have:

```
for (var key in data)
```

The variable `key` is initialised with a default value of `undefined` and then used within the loop. Using the var statement means you can initialise a variable but not set it's value until later, and yet it's value will get hoisted to the top of the function. So within my loop, the `onclick` event handler gets the hoisted value of key, which will be the last value because the handler gets assigned after all the code has run. 

## Some solutions
My first approach at a solution was to use a more general event handler, instead of assigning to each cell, I could assign a handler to the entire table and then filter that event. Something along the lines of:

```
document.querySelector('.table tbody').addEventListener('click', function(event){
         if (event.target.tagName.toLowerCase() === 'a') {
          validateFromForm(event.target.innerHTML);
         }
      });
```

This solution means that every time you click on the table anywhere in `<tbody>` there will be an event that Javascript captures, and then you can filter it as need be. This is called event delegation.

Another option is to make use of block scoping. In Javascript, block scoping is done through the (`let` statement)[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let. The simple change would be:

```
for (let key in data)
```

This results in the variable `key` become scoped to within the local block expression, meaning every iteration block. This will let the eventhandler have a different key variable each iteration. Using `const` is also an option, but the key difference is that you cannot access it before it is initialised and declared. Both `var` and `let` will be undefined when initialised.

