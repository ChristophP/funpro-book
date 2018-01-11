# API Documentation

Many of the examples contain functions with ramda. The reason is many ramda
function like `R.map` for example detect the if the passed object has a `map`
method itself and then dispatches to that. This is cool because it let's you
write `R.map(someFunc, somethingWithMap)` instead of `somethingWithMap.map(someFunc)`.
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

### .Just

Takes some value and creates a `Just`

```js
// a -> Maybe a
Maybe.Just

Maybe.Just('Nicky') // Just('Nicky')
```

### .Nothing

Creates a `Nothing`. Takes no arguments.

```js
// () -> Maybe a
Maybe.Nothing

Maybe.Nothing() // Nothing
```

### #map

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

### #chain

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

## Result

Represents a result for an operation that might fail. In Elm syntax:
```elm
type Result a b = Err a | Ok b
```
It is a Functor(aka has a `map` function) and also Monad(aka has a `chain` function).

### .Ok

Takes a value and creates an `Ok`.

```js
// a -> Result a
Result.Ok

Result.Ok(42) // Result(42)
```

### .Err

Takes a value and creates an `Err`.

```js
// a -> Result a
Result.Err

Result.Err('Oh no') // Result('Oh no')
```

### #map

Let's you transform the value inside a Result. If it is an `Ok` the function
will be applied to it. If the Result is a `Err` the map has no effect.

```js
// (a -> value) -> Result x a -> Result x value
Result.prototype.map

const toUpper = str => str.toUpperCase();
Result.Ok('Nicky').map(toUpper) // Ok('NICKY')
Result.Err('Oh no!').map(toUpper) // Err('Oh no!')

// with ramda
R.map(toUpper, Result.Ok(4));
```

### #mapError
Let's you transform the error inside a Result. If it is an `Err` the function
will be applied to it. If the Result is an `Ok` the function has no effect.

```js
// (a -> value) -> Result a x -> Result value x
Result.prototype.mapError

const toUpper = str => str.toUpperCase();
Result.Ok('Nicky').mapError(toUpper) // Ok('Nicky')
Result.Err('Oh no!').mapError(toUpper) // Err('OH NO!')
```

### #chain

Let's you chain together multiple Results, while removing one layer so you don't
end up with nested Results. The callback will receive the value of the previous
Result, if is an `Ok` or will skip if it is an `Err`.

```js
// (a -> Result value) -> Result x a -> Result x value
Result.prototype.chain

const safeParseInt = str => {
  const int = parseInt(str);
  return isNaN(int) ? Result.Err('Nope') : Result.Ok(int);
};
Result.Ok('123').chain(safeParseInt) // Ok(123)
Result.Err('This is NaN').chain(safeParseInt) // Err('This is NaN')

// with ramda
R.chain(safeParseInt, Result.Ok(4));
```

## union

Create your own union types. It takes an object, which Constructor names as
keys and the number of arguments(arity) of that the constructors can take.

```js
Name = union({
  FullName: 2,
  NickName: 1,
  NoName: 0,
});

const name1 = Name.FullName('Peter', 'Lustig');
const name2 = Name.NichName('Pete');
const name3 = Name.FullName();
```

The constructors are always curried. So if you have an arity of more than 2,
you can do partial function application.

```js
const lastNameToName = Name.FullName('Max'); // returns a function which takes the second argument.
const name = lastNameToName('Pearson'); // Name('Max', 'Pearson')
```

You can extend the type by accessing the prototype and adding methods, like this.
```js
Name.prototype.toString = function() {
  return matchWith(this, {
    FullName: (first, last) => `${first} ${last}`,
    NickName: name => name,
    NoName: 'Unknown',
  });
}
```

### matchWith

Pattern match on union types. Takes a union and an object of cases. The keys
need to be the constructors of the union and the values functions which accept
the arguments stored within each constructor. It returns the return value of the
function which is matched.

```js
const age = Just(30);
matchWith(age, {
  Just: val => val.toString(),
  Nothing: () => 'I am not saying',
}); // returns '30'


const result = Result.Err('Bad Error');
matchWith(result, {
  Ok: val => val.toString(),
  Err: err => `Geez, not again: ${err}`,
}); // returns 'Geez, not again: Bad Error'
```

Errors are thrown when:
- The first argument is not a union.
- Not all cases are handled in the case object.
- Cases are handled that don't correspond to any constructor.

```js
// throws an error because Nothing is not handled
matchWith(age, {
  Just: val => val.toString(),
});

// throws an error because Nuttin is not a recognized pattern for `Maybe`
matchWith(age, {
  Just: val => val.toString(),
  Nuttin: () => 'I am not saying',
});

// throws an error because the first argument is not a union
matchWith({}, {
  Just: val => val.toString(),
  Nothing: () => 'I am not saying',
});
```

## Task

Task are there for asynchronous tasks and to wrap up side-effects.
They compose pretty well and can be mapped and chain.

In elm code:
```elm
Task error value
```

### .of

Creating a Task from a function and a list of arguments it should be called
with. The function can return a regular value or a Promise.

```js
// get from the network in the browser
Task.of(fetch, ['https://example.com', { credentials: 'include' }]);

// get a random number
Task.of(Math.random);

// set a timeout
Task.of(() => new Promise((resolve) => setTimeout(resolve, 1000)));
```

### .succeed

Create a task that immediately succeeds with the given value.

```js
// get from the network in the browser
Task.succeed(42);
```

### .fail

Create a task that immediately fails with the given error. The error can be any
value but should ideally be a string or an Error object.

```js
// get from the network in the browser
Task.fail('Something went wrong');

// get from the network in the browser
Task.fail(new Error('OMG'));
```

### .all

Takes a list of tasks and returns a new task which, will (when run) execute
all of these tasks concurrently. If successful it will succeed with an array
of the tasks results. The first task that fails will make everything fail.

```js
Task.all([Task.of(fetch, ['my.api.com']), Task.of(fetch, ['example.com'])])
  .map(([result1, result2]) => ...) // will be mapped if fetch succeeds
  .mapError(err => ...) // will be called if fetch fails

Task.all([Task.succeed(42), Task.fail('fail one'), Task.fail('fail 2')])
  .mapError(err => ...) // 'fail one'
```

### .sequence

Like `Task.all` but executes the tasks sequentially.

```js
Task.sequence([Task.of(fetch, ['my.api.com']), Task.of(fetch, ['example.com'])])
  .map(([result1, result2]) => ...) // will be mapped if fetch succeeds
  .mapError(err => ...) // will be called if fetch fails

Task.sequence([Task.succeed(42), Task.fail('fail one'), Task.fail('fail 2')])
  .mapError(err => ...) // 'fail one'
```
### #map

Let's you transform the value with which the Task will resolve.
If the task succeed the function will be applied to it its value.
If it fails the map has no effect. `map` returns a new Task with the mapped value.
The mapping will be executed as soon as the task succeeds.

```js
// (a -> value) -> Task x a -> Task x value
Task.prototype.map

const toUpper = str => str.toUpperCase();
Task.succeed('Hello').map(toUpper) // will map the value
Task.fail('Oh no').map(toUpper) // does nothing

// with ramda
R.map(toUpper, Task.succeed('Hello'))
```

### #mapError

Let's you transform the value with which the Task will resolve.
If the task succeed the function will be applied to it its value.
If it fails the map has no effect. `map` returns a new Task with the mapped value.

```js
// (a -> b) -> Task a value -> Task b value
Task.prototype.mapError

const toUpper = str => str.toUpperCase();
Task.succeed('Hello').map(toUpper) // does nothing
Task.fail('Oh no').map(toUpper) // will map the error
```

### #chain

Chain multiple tasks together. The first tasks will be completely done, when
the second task starts. Takes a function which gets the resolved value of the
previous task and returns the next task. The chain will end as soon as one task
fails.

```js
// (a -> Task err value) -> Task x a -> Task err value
Task.prototype.chain

const authTask = Task.of(fetch, ['auth.api.com']);
const loadDataTask = token => Task.of(fetch, [api.data.com]);
Task.of(authTask).chain(loadDataTask) // chains the two tasks

// you can chain as many as you want
Task.of(authTask)
  .chain(loadDataTask)
  .chain(data => someOtherTask) // can go on and on

// with ramda
R.chain(token => Task.of(fetch, [api.data.com]), Task.of(fetch, ['auth.api.com']))
```

### #onError

Recover from failing tasks. Takes a function, which receives the error and returns
a new task.

```js
// (err -> Task x value) -> Task err a -> Task x value
Task.prototype.onError

Task.fail('Oh no').onError(err => Task.of(fetch, ['other.data.com'])) // will rescue use the fallback
Task.succeed('Hello').onError(err => Task.of(fetch, ['other.data.com']) // does nothing
```

### #run

This is how tasks are actually executed. Since calling this function is **impure**
you should ideally only call it once in you entire program. If you `chain` and
`map` correctly then more than once shouldn't be necessary and keeps the rest of your
code **pure**. It returns a promise which fulfills if the Task succeeds or rejects
if the task fails. If you use `run` right and only use it to kick of your application
then you can typically ignore this promise.

```js
Task.prototype.run

Task.of(fetch, ['api.com']).run(); // actually runs the network request

// this is how create a task and only need one `run` with pure code
const main = Task.of(authTask, ['someid'])
  .chain(token => dataTaskFromToken(token))
  .map(data => ...)
  .onError(err => ...)

// impure code
main.run();
```
