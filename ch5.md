# API Documentation

Many of the examples contain functions with ramda. The reason is many ramda
function like `R.map` for example detect the if the passed object has a `map`
method itself and then dispatches to that. This is cool because it let's you
write `R.map(someFunc, somethingWithMap)` instead of somethingWithMap(someFunc).
and let's you take advantage of ramda pre-curried functions and let you apply
the function partially. LINK. This is in line with fantasyland LINK and you
can pretty mcuh expect this behaviour with anything that has a `map` function
(aka Functors), `ap` function (aka Applicatives), `chain` function (Monads) and
and `concat` function (Monoid).

## Maybe

Represents a value that might not be there. In Elm syntax:
```elm
type Maybe a = Just a | Nothing
```
It is a Functor(aka has a `map` function) and also Monad(aka has a `chain` function).

### map

Let's you transform the value inside a Maybe. If it is a `Just` the function
will be applied to it. If the Maybe is a `Nothing` the map has no effect.

```js
// (a -> b) -> Maybe a -> Maybe b
Maybe.prototype.map

const toUpper = str => str.toUpperCase();
Maybe.Just('Nicky').map(toUpper) // Just('NICKY')
Maybe.Nothing().map(toUpper) // Nothing

// with ramda
R.map(toUpper, Maybe.Just(4))
```

### chain

Let's you chain together multiple Maybes, while removing one layer so you don't
end up with nested Maybes. The callback will receive the value of the previous
Maybe, if is a `Just` or will skip if it is a `Nothing`.

```js
// (a -> Maybe b) -> Maybe a -> Maybe b
Maybe.prototype.chain

const listHead = list => list.length === 0 ? Maybe.Nothing() : Maybe.Just(list[0])
Maybe.Just([1, 2, 3]).chain(listHead) // Just(1)
Maybe.Nothing().chain(listHead) // Nothing

// with ramda
R.chain(listHead, Maybe.Just(4))
```

