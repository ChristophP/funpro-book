# Deedo's tips for FP with JS

Douglas Crockford was one of the earliest to say that, JS has good
and bad parts and using only a subset of the features, actually
makes the language more pleasant to use. He dubbed it "The Good Parts".

Software is often deemed better the more it can do. However with languages
sometimes that is not true. Elm also follows the
principle of making things better by removing things and focusing
on good ways to do it rather than many ways to do it. While JS
certanly can do a lot, I am a strong believer that it is better
to pick a certain paradigm of programming and then stick to a couple
of features that let you get the job done in an efficient way.
So here are my "Good Parts of functional JS", which I hope will guide
you in your functional journey.

## Don't use `let` and use `const` instead.

With the arrival of ES6 the `var` keyword got some new buddies named `let` and `const`.
Using `const` is great because it makes you program very simple, since you can
assign `const` variables only once. With `let` or `var` that isn't the case:

```js
const val = 3; // will always be three

let val2 = 6; // can be reassigned
var val3 = 6; // can be reassigned
```

With `const` you never have to check if that reference it changed, which makes
the program very easy and straighforward.

## Avoid statements and try to use expressions

This one goes a long way. The difference between a statement and an expression is
that an expression can always be evaluated to a single value. Just use this example
about two ways to create a conditional.

### Use the ternary operator over an if
```js
// the expression is evaluated and gives you a value.
const val = someCondition ? 3 : 6;

// with a statement you cannot use a constant
let val;
if (someCondition) {
  val = 3;
} else {
  val = 6;
}
```

### Avoid loops, use functions and recursion

## Use arrow functions and currying

As you probably know there are two kinds of functions in JS. The ones which are
created with the `function` keyword and the ES6 addition arrow functions.

So why shouldn't you use the former? Those functions are more suited to do
OOP. They define a `this` which points to the object on, which the method was called.

```js
const obj = {};
obj.someMethod = function() { console.log(this); };
obj.someMethod(); // logs `this`, which points to `ob`
someMethod(); // no object here, this points to the `window` or global object
```

Since we want be pure in FP, we do not want that reference to `this`, using it
breaks the purity by which a functions output must solely depend on it's arguments.

It's probably best to avoid `this` altogether. Using arrow functions is a
good practice that helps that cause because they don't define their own `this`.
Unfortunately you can still use `this` in them, but it will simply be the same
reference as the one of the outer scope.

```js
const obj = {};
obj.someFunction = () => this;
obj.someFunction(); // returns `this`, which points to `window` or global object
someFunction(); // also returns `this`, which points to `window` or global object
```

## Currying and partial function application

- Haskell Curry origins
- simple arrow currying
- generic curry functions

Use these 2 terms in sentence: Okay I got it. Currying is a process, to transform
a function into another function which can be partially applied.

So far so good. But what is partial function application? Most JS developers
are used to passing all arguments to a function at once. But in some FP
languages (i.e. Elm, Haskell) it is okay to pass only some of the arguments and
get a function back that accepts the remaining arguments. This concept is also
used by `Redux`'s connect function for example.

```js
import { connect } from 'redux';

...

const connectedComponent = connect(mapStateToProps)(MyComponent);
```

`connect` accepts some mapping function and returns another function whcih is then
called with the component which then returns the final component which is
connected to the redux store.

Let's look at another example.

```js
const add = a => b => a + b;

const add3 = add(3);

add3(10); // 13
```

We easily created a curried function just by using arrow functions.

However, this means also means that if you do want to pass all arguments at the
same time you have to do this `add(3)(10)` which most people would fine annoying
and this also requires you to know which function is curried in which way and so on.

Let's fix this. We can use JS to create a function, which curries other functions
in just a couple of lines.

```js
const curry = (fn) => {
  const resolver = (...args) => {
    return (...innerArgs) => {
      const local = [...args, ...innerArgs];
      return local.length >= fn.length ? fn(...local) : resolver(...local);
    };
  };
  return resolver();
};
```

The resulting function will check how many arguments are passed and return another
function expecting the remaining arguments or the fonal result if all arguments
where there. Let's do this again with add.

```js
const add = (a, b) => a + b; // not a curried function
const curriedAdd = curry(add);

curriedAdd(3, 6) // 9
curriedAdd(3)(6) // 9
```

`Ramda`'s function are all precurried so you can call them passing all arguments
or only a couple at a time.


## Modelling data accurately
