# Deedo's tips for FP with JS

Douglas Crockford was one of the earliest to say that, JS has good
and bad parts and using only a subset of the features, actually
makes the language more pleasant to use. He dubbed "The Good Parts".

Software is often deemed better the more it can do. Elm also follows the
principle of making things better by removing things and focusing
on good ways to do it rather than many ways to do it. While JS
certanly can do a lot, I am a strong believer that it is better
to pick a certain paradigm of programming and then stick to a couple
of features that let you get the job done in an efficient way.
So here are my "Good Parts of functional JS", which I hope will guide
you in your functional journey.

- no let, use const instead (what about var?)
- avoid statements and try to use expressions
- no loop, use functions and recursion
- use arrow functions and currying
  - Haskell Curry origins
  - simple arrow currying
  - generic curry functions
- modeling data accurately
