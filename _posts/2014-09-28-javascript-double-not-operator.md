---
layout: post
title: Javascript Double Not-Operator(!!)
categories:
- Javascript
tags:
- js
---

In javascript Falsy and Truthy values are used all over the place.

For example the following values are always falsy:

- `""` (empty string)
- `undefined`
- `false`
- `0`
- `null`
- `NaN`

And any other value is truthy.

Since undefined and empty string are falsy values then we could replace this code below:

```js
if (typeof x !=== 'undefined' && x !=== '') {
  // not emtpy
}
```

with a falsy/truthy verification:

```js
if (x) {
  // not emtpy
}
```

To set the result of falsy/thruthy verification to a variable is possible to use a double not-operator(`!!`) which will force a type casting to Boolean.

```js
var xIsEmpty = !!x;
```

Actually `!!` is not a operator it is only the `!` operator twice. The first `!` will cast to boolean inverting the result. Then the second `!` will invert again the value so it is the expected boolean value.

```js
var x = 0;        // falsy
console.log(!x);  // true   (it is not what we want)
console.log(!!x); // false
```
