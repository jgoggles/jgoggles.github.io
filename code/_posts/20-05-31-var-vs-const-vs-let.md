---
title: "The Differences Between var, const, and let"
---

Since the introduction of ES6's `const` and `let`, something as simple as variable assignment can cause a bit of confusion in JavaScript.
But doing a quick investigation of their differences can also teach us some interesting things about scope in JavaScript, a key concept in any language.

## TLDR
- `var` is function scoped and its variables can be accessed before they're declared (returns `undefined`)
- `let` is block scoped and will return a `ReferenceError` if a variable is accessed before it's declared
- `const` is block scoped and will return a `ReferenceError` if a variable is accessed before it's declared; cannot be reassigned


## `var`

### Scope

First let's look at `var`. The `var` declaration should be familiar to all JavaScript developers. You want to declare a variable?

```JavaScript
var foo = "bar"
```

Boom. Done. Right?

Pretty much, yes. But there's a bit more to that basic assignment going on under the hood and it has to do with scope. Scope is basically the context that your variables live inside of.
When declaring a variable with `var` JavaScript will make that variable available within the nearest function declaration. In other words, `var` is _function scoped_.

For example:

```JavaScript
function foo () {
  var bar = 'baz'
  console.log(bar) // baz
}

console.log(bar) // Uncaught ReferenceError: bar is not defined
```

However, variables defined with `var` _will_ be available to nested functions and returned methods:

```JavaScript
function foo () {
  var bar = "baz"

  return {
    fiz () {
      console.log(bar)
    }
  }
}

console.log(foo().fiz()) // bar
```

Function scoping can be a little weird. Here's why:

```JavaScript
function favoriteFriends (friends) {
  for (var i = 0; i < friends.length; i++) {
    var friendName = friends[i]
    var rank = i + 1
    console.log(`${friendName} is my #${rank} friend`)
  }

  console.log(i)
  console.log(friendName)
  console.log(rank)
}

favoriteFriends(["Charlie", "Michelle", "Curtis"])
// Charlie is my #1 friend
// Michelle is my #2 friend
// Curtis is my #3 friend
// 3
// Curtis
// 3
```

Since a `for` loop is just a function all of its variables are available in the outer function. Not necessarily what you expect or want. 

### Declaration vs Initialization

`var` variables also use JavaScript's standard initialization and declaration rules. Let's see what that means:

```JavaScript
var foo // this is called initialization
console.log(foo) // undefined

foo = "bar" // this is the declaration
console.log(foo) // bar

```

Variables are initialized with a value of `undefined` when they're created. Once we assign a value to the variable that's called decalaration. Normally we do the initialization and
declaration in the same step (`var foo = "bar"`), but when our code is interpereted JavaScript does something called _hoisting_. Let's take a look:

```JavaScript
// When we do this
function favoriteFriends (friends) {
  for (var i = 0; i < friends.length; i++) {
    var friendName = friends[i]
    var rank = i + 1
    console.log(`${friendName} is my #${rank} friend`)
  }
}

// it gets interpereted as this; this is hoisting
function favoriteFriends (friends) {
  var i = undefined
  var friendName = undefined
  var rank = undefined

  for (i = 0; i < friends.length; i++) {
    friendName = friends[i]
    rank = i + 1
    console.log(`${friendName} is my #${rank} friend`)
  }
}
```

That's why we're able to `console.log` a variable before we actually declare it. 

## `const & let`

This brings us finally to the two newer kids on the block, `const` and `let`. The differences are pretty simple.

### Scope

Unlike `var` which is function scoped, `const` and `let` are both _block scoped_. Let's look back at our `favoriteFriends` function using `let` instead:

```JavaScript
function favoriteFriends (friends) {
  for (let i = 0; i < friends.length; i++) {
    let friendName = friends[i]
    let rank = i + 1
    console.log(`${friendName} is my #${rank} friend`)
  }

  console.log(i)
  console.log(friendName)
  console.log(rank)
}

favoriteFriends(["Charlie", "Michelle", "Curtis"])
// Charlie is my #1 friend
// Michelle is my #2 friend
// Curtis is my #3 friend
// Uncaught ReferenceError: i is not defined
```

Since `let` is block scoped it's only available within the block of code it's used inside of. You can think of a block as the nearest containing curlies (`{}`).

### Initialization

The second main difference between `var` and `const` and `let` is how they are initialized. Remember `var` gets hoisted and initialized with a value of `undefined`:

```JavaScript
console.log(foo); var foo = "bar"
// undefined
```

But that doesn't work with `const` and `let`:
```JavaScript
console.log(foo); const foo = "bar"
//Uncaught ReferenceError: Cannot access 'foo' before initialization

console.log(fiz); let fiz = "baz"
//Uncaught ReferenceError: Cannot access 'fiz' before initialization
```

## `const vs let`

The only difference between `const` and `let` is that `let` values can be reassigned and `const` values cannot.

```JavaScript
const foo = "bar"
foo = "biz"
//Uncaught TypeError: Assignment to constant variable. 🙅🏻‍♂️

let foo = "bar"
foo = "biz"
"biz"
// 👍🏼
```

## Tell me what to use

There is no single right answer. In general I always use `const` unless I know the value will change (like inside a loop), then I'll use `let`.

## Conclusion

Hopefully that gives some context around the differences between `var`, `const`, and `let` plus a peek into how JavaScript treats them internally.
