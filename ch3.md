# Async and IO

As functional programmers our goal is to keep our code free of side effects and
handle async stuff in a way that doesn't break our precious pure functions.

My favorite ES6 feature has always been Promises. Using them at first was just
mindblowing. Seeing the callback hell freeze over and handling async with elegence. They
compose very well and can also smoothly be blended together one with `Promise.all()`.

However, for functional programming, Promises possess some subpar properties.
Number 1, they are not pure. Promises contain some interal state, which indicating
if it is pending, fulfilled or rejected, which is mutated over the course of
its lifetime. Another biggie is that the creation of a Promise alone
already triggers some async action.

So the trick is to use Tasks for anything that breaks the laws of purity.

1. Same input will always generate the same output.
2. No side-effects as a result of running the function.

So by that logic, you should use a task for stuff like getting a random number,
writing a file, making a network call, setting a timer and so on and also for
anything that works asynchronously.

## Creating and combining

Like I mentioned, Promises execute right away. So Tasks use a trick of wrapping
Promises into functions so the execution can be delayed till later.

```js
// this would fire of an http request
fetch('/some/url');

// but this won't until the function is called
() => fetch('/some/url');
```

You can create a Task with `Task.of`. It takes a function as the first argument
and a list of parameters with which the function will be called as the second one.
The function can return a promise or any normal value.

```js
Task.of(fetch, ['/some/url']);

Task.of(Math.random);

Task.of(time => new Promise((resolve) => { setTimeout(resolve, time) }), [4000]);
```

Great, now we packaged up our impure code in the Task. Next we need to run it
from a safe location.

## Running tasks

We can't do impure stuff without tainting some places of our code, the good news
is that you can do as many impure things as you want and only have one impure
line of code. All task have a `run` method, which actually execute the function
that your task contains.

```js
someTask.run();
```

You can then combine your task with other tasks to build your entire app, using
`map` and `chain`. This is an example of a little program, which generate a random number,
hashes the number to create an id,
fetches something from the network, get's the result and writes it to a file
before printing that it's done.


```js
// define a bunch of function which create tasks
const getRandom = Task.of(Math.random);
const fetchData = id => Task.of(fetch, [`somedomain.com?id=${id}`])
  .chain(res => Task.of(res.text);
const writeFile = (name, content) =>
  Task.of(writeFileAsync, [name, content, 'utf8']);
const print = str => Task.of(console.log, [str]);

// export a Task so the importing module can call .run()
module.exports = getRandom
  .map(md5)
  .chain(fetchData)
  .chain(content => writeFile('pure-test.file', content))
  .chain(() => print('Done'));
```

By using Tasks that don't execute right away, you are giving the control over
the side effects to the calling program. It's like handing somebody a grenade
and letting them pull the pin out. This way you keep your hands clean and stay
pure and push the impure code to the edges of you application. This is very
similar to the IO Monad in Haskell that only execute if it is passed to the
`main` function of the program or contain within another IO monad.
In JS we have to pull the trigger ourselves,
ideally only calling `.run()` once in the entire program. That way, you can
write your entire program without impure code except for that one line and your
program will be easy to test and simple to reason about.

## Working with tasks

Mapping allows you to access the value the task resolves to once it is run
and transforming it without taking it out of the Task context. Chaining also
allows you to access that value, but lets you chain new tasks onto the previous
one. But to make tasks truly powerful, we need a way to handle errors and
control execution flow.

```js
Task.fail('Oh no').mapError(str => str.toUpperCase())
```
`Task.fail` creates a task that immediately fails with an error. We can then use
`mapError` to map the error. If the tasks succeeds `mapError` does nothing.
Maping the error does NOT make the task succeed. We can use `onError` for that.

```js
Task.fail('Oh no')
  .map(() => 'This is never called')
  .onError(_ => Task.succeed('Default value'))
```

This actually lets you catch a failing task and gives you the power to turn
it into a new task. Failing task will skip `map` and `chain`, and jump to the next
`onError` or `mapError`. The reverse is true for succeeding tasks, so `map` and
`chain` away knowing it will only get called if the time is right. This is
pretty similar to Promises and their `then` and `catch` method.

Sometimes you might also wanna control the execution flow of multiple tasks.
For concurrent task you can use `Task.all`. This works pretty much like
`Promise.all`. It returns a new task, which resolves with an array of
the result of all tasks in the list.

```js
// will be run concurrently
Task.all([fetchTask, someOtherFetchTask])
  .map(([result1, result2]) => ...);
```
For running things sequentially we can use `Task.sequence`.

```js
// will be run sequentially
Task.sequence([fetchTask, someOtherFetchTask])
  .map(([result1, result2]) => ...);
```

This is actually just a short hand for this:
```js
fetchTask.chain(() => someOtherFetchTask)
  .map(([result1, result2]) => ...);
```

## Summary

Keeping you code pure is one of the main goals of FP because it helps you
write clean, predictable, easy-to-test code. Using tasks inverts the control
for executing side-effects and async operation, form the called program to the
calling program. If you use it right, your entire program will only have one
`run()` call. That way if weird things happen. you will know,
where to look for quickly.
