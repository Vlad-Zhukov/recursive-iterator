# Recursive Iterator

## About
Iterates javascript object recursively.
Works in ES5 and ES6 environments.
Supports ES6 [iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols).
Compatible with [for...of](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/for...of) cycle.

## Required
ES5 Object.keys(), Array.prototype.indexOf()

## Syntax
```js
var iterator = new RecursiveIterator(
    root /*{Object|Array}*/,
    [bypassMode=0] /*{Number}*/,
    [ignoreCircular=false] /*{Boolean}*/,
    [maxDeep=100] /*{Number}*/
);

var item = iterator.next();
var state = item.value; // descriptor of state
var done = item.done; // true if end of the loop

var parent = state.parent; // parent object
var node = state.node; // current node
var key = state.key; // key of node
var path = state.path; // path to node
var deep = state.deep; // current deep
```

## Tree traverse methods
By default method bypass of tree is vertical `bypassMode=0`:
```js
var root = {
    object: {
        number: 1
    },
    string: 'foo'
};

var iterator = new RecursiveIterator(root);
for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// object    Object {number: 1}
// object.number    1
// string    foo
```
You can change it on horizontal by passing the `bypassMode=1`:
```js
var root = {
    object: {
        number: 1
    },
    string: 'foo'
};

for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// object    Object {number: 1}
// string    foo
// object.number    1
```

## Circular references
By default, if detected circular reference then will throw an exception:
```js
var root = {
    number: 1,
    object: undefined,
    string: 'foo'
};
root.object = root;

var iterator = new RecursiveIterator(root);
for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// number    1
// Uncaught Error: Circular reference
```
You can change this behaviour by passing `ignoreCircular=true`:
```js
var root = {
    number: 1,
    object: undefined,
    string: 'foo'
};
root.object = root;

var iterator = new RecursiveIterator(root, 0, true);
for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// number    1
// string    foo
```

## Max deep
You can control the depth of dives of the iterator.
By default `maxDeep`is `100`. Minimum depth is `1`.
Each like cycle:
```js
var root = {
    object: {
        number: 1
    },
    string: 'foo'
};

var iterator = new RecursiveIterator(root, 0, false, 1);
for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// object    Object {number: 1}
// string    foo
```

## Methods

### next()
Returns next [state object](https://github.com/nervgh/recursive-iterator#syntax) or `undefined`.

### isLeaf(node)
Returns `true` if node is leaf.
Leaf is all primitive types and objects whose `keys.length === 0`.
```js
var iterator = new RecursiveIterator(root);
var item = iterator.next();

iterator.isLeaf(item.node);
```

### isCircular(object)
Returns `true` if object is circular reference.
```js
var iterator = new RecursiveIterator(root);
var item = iterator.next();

iterator.isCircular(item.node);
```

### destroy()
Clears the cache and queue. It calls automatically at the end of the loop.

## Callbacks

### onStepInto(object)
It calls for each object. If returns `false` object will be skipped:
```js
var root = {
    object: {
        number: 1
    },
    string: 'foo'
};

var iterator = new RecursiveIterator(root);
iterator.onStepInto = function(object) {
    // prevent step into object
    return false;
};

for(var item = iterator.next(); !item.done; item = iterator.next()) {
    var state = item.value;
    console.log(state.path.join('.'), state.value);
}

// object    Object {number: 1}
// string    foo
```

## ES6 example: **for...of** loop
```js
var root = {
    object: {
        number: 1
    },
    string: 'foo'
};

for(let item of new RecursiveIterator(root)) {
    console.log(item.path.join('.'), item.node);
}

// or

for(let {parent, node, key, path, deep} of new RecursiveIterator(root)) {
    console.log(path.join('.'), node);
}

// or

for(let {node, path} of new RecursiveIterator(root, 1)) {
    console.log(path.join('.'), node);
}
```