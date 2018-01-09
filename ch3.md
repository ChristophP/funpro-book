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

So by that logic you should use a task for stuff like getting a random number,
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
and a list of parameters as the second one. The function can return a promise or
any normal value.

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
`map` and `chain`. This is an example of a little programm which generates an id,
fetches something from the network, get's the result and writes it to a file
before printing that it's done.


```js
const generateId = Task.of(Math.random);
const fetchGoogle = num => {
  return Task.of(fetch, [`google.com?${num}`]).chain(res =>
    Task.of(() => res.text())
  );
};
const writeFile = (name, content) =>
  Task.of(writeFileAsync, [name, content, 'utf8']);
const print = str => Task.of(console.log, [str]);

// get random number
// make request to google
// write file
// print finish

// export a Task
module.exports = generateId
  .chain(fetchGoogle)
  .chain(content => writeFile('pure-test.file', content))
.chain(() => print('Done'));``js
```

## Batching tasks
sequence, all
