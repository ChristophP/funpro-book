# Async and IO

Write stuff about tasks, how they're more converniant and flexable than promises
and how you cna use them to push side-effects to the border of your program and
handle async stuff elegantly.

My favorite ES6 feature has always Promises. Using them was just mindblowing.
Seeing the callback hell freeze over and handling async with elegence. They
compose very well and can also easily blended together one with `Promise.all()`.

However, for functional programming Promises possess some subpar properties.
Number 1 they are not pure. Promises contain some interal state, which indicating
if it is pending, fulfilled or rejected, which is mutated over the course of
its lifetime. Another biggie is that the creation of a Promise alone
already triggers some async action.

## Creating tasks


## Running tasks


