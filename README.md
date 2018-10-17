# Alternatives to `if`

This is a collection of alternatives to using `if` and other conditional block statements in Javascript.

Some reasons to consider alternatives:

 * boost immutability: because `const` is scope-level, it is unusable in functions using `if` except within the blocks -- for assignment, `let` or `var` are the only options, which are mutable
 * quality of code life: `if` is block statement that has a tendency to beget nesting block statements ad infinitum, and become progressively harder to read as they are added to
 * avoid refactoring later: many `if`s can usually indicate you really need to break your functions down further, and that your function is doing too much at once
 * often it should really just be a function: because it is a block-level element, the block can easily grow into something that clearly should be a function, or even many functions
 * control side-effects: `switch` statements can easily "leak" when missing a `break` statement, and create "branches" of code without braces, hiding these branches somewhat
 * testability: each `if` creates a logical branch that cannot be independantly tested
 * functional: if functional programming is something you want to do more of, you'll generally want less `if` and `let`
 * be safer with type: when you convert an `if` into a function, you can test for types in TypeScript and Flow
 
# Caveat

The following examples may or may not adhere to a _functional programming_ style, but can be used as a stepping stone towards more functional code.

# Kinds of ifs

Each example is a very basic form of a common conditional pattern. These suggestions are meant to give you some ideas of alternatives.

Some examples, such as the "incongruent `if`/`else`" examples could be considered antipatterns and are themselves likely to be considered by most as poorly-formed `if` statements.

## Basic `if`

```js
let y;
if (x === 1) {
    y = 2;
}
```

### Alternatives

#### Ternary assignment

Basic, returns `y` as `false` for default.

```js
// ternary with false default
const y = x === 1 ? 2 : false;
```

#### Functional ternary

Similar, but has a reusable function that can be memoized.

```js
// functional ternary
const checkX = x => (x === 1 ? 2 : false);
const y = checkX(x);
```

## `if`/`else`

```js
let y;
if (x === 1) {
    y = 2;
} else {
    y = 1;
}
```

### Alternatives

#### Ternary assignment

Very basic, readable. Allows `y` to be a `const`.

```js
// ternary assignment
const y = x === 1 ? 2 : 1;
```

#### Functional ternary

Very similar, but reusable and [memoizable](https://medium.freecodecamp.org/understanding-memoize-in-javascript-51d07d19430e).

```js
// functional ternary
const checkX = x => (x === 1 ? 2 : 1);
const y = checkX(x);
```

## `if`/`else`/`else if`

```js
let y;
if (x === 1) {
    y = 2;
} else if (x === 2) {
    y = 17.3;
} else {
    y = 1;
}
```

## or, `switch`/`case`/`break`

```js
switch (x) {
    case 1:
        y = 2;
        break;
    case 2:
        y = 17.3;
        break;
    default:
        y = 1;
}
```

### Alternatives

#### Object key-value pairs

Use an object to store values as key-value pairs.

```js
// key-value pair list with default
const vals = {
    1: 2,
    2: 17.3,
    default: 1
};
const y = x in vals ? vals[x] : vals.default;
```

#### Object key-value pairs with default symbol

Very similar, and prevents namespace collisions: use a symbol for your default value key. (The example above with `default` key would likely misbehave if `x` ever had a value called `default`.)

```js
// key-value pair list with symbol
// this avoids namespace collision in key names
const $$defaultSymbol = Symbol();
const vals = {
    1: 2,
    2: 17.3,
    [$$defaultSymbol]: 1
};
const y = x in vals ? vals[x] : vals[$$defaultSymbol];
```

#### Array-based value list

Or, use an array to store the values. Works only if dealing with low numbers.

```js
// value list accessed by index
const vals = [null, 2, 17.3];
const y = vals[x] || 1;
```

#### Constant function expression getter

Use a getter inside a constant function expression as a means of looking up a key-value pair. Allows for memoization, though uses the dreaded `this` keyword...

```js
// getter
const vals = x => ({
    1: 2,
    2: 17.3,
    get value() {
        return this[x] || 1;
    }
});
const y = vals(x).value;
```

#### Conditionals table

Constant function expression returning a lookup table: zeroth index is the condition, first index is the result.

```js
// conditionals table
const condTable = x => [
    [x === 1, 2],
    [x === 2, 17.3],
    [true, 1]
];
const y = condTable(x)
    .find(a => (a[0] ? a[1] : false))
    .pop();
```

## Incongruent `if`/`else if` conditions

```js
let y;
if (x === 1) {
    y = 2;
} else if (x === 2) {
    y = 17.3;
} else if (z === 4) {
    y = 19.2;
{ else if (w === 'something') {
    y = 22;
} else {
    y = 1;
}
```

### Alternatives

#### Conditionals table

Another constant function expression returning a lookup table, this time taking an array as a parameter. The array param allows for memoization of the table.

```js
// conditionals table with array param
// array-based param allows for memoization if need be
const condTable = ([x, z, w]) => [
    [x === 1, 2],
    [x === 2, 17.3],
    [z === 4, 19.2],
    [w === 'something', 22],
    [true, 1]
];
const y = condTable([x, z, w])
    .find(a => (a[0] ? a[1] : false))
    .pop();
```

## Incongruent `if`/`else if` results

What if not just the conditions are mixed, but the results too?

```js
let y;
let j;
if (x === 1) {
    y = 2;
} else if (x === 2) {
    y = 17.3;
} else if (z === 4) {
    y = 19.2;
{ else if (w === 'something') {
    y = 22;
{ else if (w === 'something else') {
    j = 5;
} else {
    y = 1;
}
```

### Alternatives

#### Conditional functions table

Similar to conditionals table, use the table to execute functions rather than assign value.

```js
// results are functions, pass everything into params that might be needed
let y, j;
const setY = v => y = v;
const setJ = v => j = v;
const condTable = ([x, z, w]) => [
    [x === 1, setY, 2],
    [x === 2, setY, 17.3],
    [z === 4, setY, 19.2],
    [w === 'something', setY, 22],
    [w === 'something else', setJ, 5],
    [true, setY, 1]
];

condTable([x, z, w])
    .find(a => (a[0] ? a[1](a[2]) : false));
```

## Nested `if`

```js
let y, z;
if (x === 1) {
    y = 2;
    if (w > 4) {
        z = 1;
    }
} else {
    y = 1;
    if (w < 4) {
        z = 2;
    }
}
```

### Alternatives

#### Functional flattening

Take each layer and transpose each `if` to a function, or assignment.

```js
const fn1 = () => {
    y = 2;
    z = w > 4 ? 1 : z;
};

const fn2 = () => {
    y = 1;
    z = w < 4 ? 2 : z;
};

const assignVals = x =>
    x === 1
        ? fn1()
        : fn2();
```
