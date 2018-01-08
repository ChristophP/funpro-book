# API Documentation

## Maybe

Represents a value that might not be there. In Elm syntax.
`type Maybe a = Just a | Nothing`
It is a Functor(aka has a `map` function) and also Monad(aka has a `chain` function)

```js
// (a -> b) -> Maybe a -> Maybe b
Maybe.prototype.map

const toUpper = str => str.toUpperCase();
Maybe.Just('Nicky').map(toUpper) // Just('NICKY')
Maybe.Nothing().map(toUpper) // Nothing

// with ramda
R.map(toUpper, Maybe.Just(4))

// (a -> Maybe b) -> Maybe a -> Maybe b
Maybe.prototype.chain

const listHead => list => list.length === 0 ? Maybe.Nothing() : Maybe.Just(list[0])
Maybe.Just([1, 2, 3]).chain(listHead) // Just(1)
Maybe.Nothing().chain(listHead) // Nothing

// with ramda
R.chain(toUpper, Maybe.Just(4))
```

