# Maybe, Result and Pattern matching

Suppose your app state needs some state to hold user information.
If the user is logged in the state should be "Authenticated" and
you also need some data about the user. Otherwise just the info
that you are "SignedOut" is enough.

Often times this scenario is modelled with a boolean and some data like this:
```js
{
  authenticated: true,
  userData: {name: 'Peter', ... }
}
```

But what if you end up in some weird scenario, where `authenticated` is true,
but the `userData` is missing, null or an empty object? Very confusing stuff,
my head is spinning so we better take great care to keep the two of them in sync.

Alright, take a breath. But what if we need to add third state for guest users?
By this time it should be pretty obvious that booleans aren't very good at
representing this type of logic. It's not just me, other people think that too.
There are some talks about this....

So let't try again by definging a union type.

```js
import { union } from 'funpro';

const Auth = union({
  Authenticated: 1,
  SignedOut: 0,
})
```

We declared a new type called `Auth` with two possible states.
`Authenticated` carries one argument, while `SignedOut` does not have one.
We can create these states by using the constructors, like this:

```js
const auth = Auth.Authenticated({ name: 'Peter', ... });

const someOtherAuth = Auth.SignedOut();
```

This is great. Now `Authenticated` comes batteries-included with the data that it needs,
while `SignedOut` is fine without any arguments. Now what about that third state
we talked about earlier. Let's assume guest accounts are for a limited
time only and have an expiration date.

```js
const Auth = union({
  Authenticated: 1,
  SignedOut: 0,
  Guest: 1,
})

const auth = Auth.Guest(new Date('20-01-2018'))
```

This was easy, now we added that third state in a breeze. We even made it carry
that extra data it so desparately needs.
