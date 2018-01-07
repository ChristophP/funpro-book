# Maybe, Result and Pattern matching

## Optional Data

```js
import { Maybe } from 'funpro';

maybeNumber = Maybe.Just(42);
```

```js
// returns a Just a price or Nothing if it isn't on sale
const maybe = item.getSalePrice();

const displaySale = matchWith(maybe, {
  Just: price => `${price} EUR`,
  Nothing: () => `not on sale`,
});
```

## Danger Zone

## Safe functions

## Methods

- map
- chain
- of
