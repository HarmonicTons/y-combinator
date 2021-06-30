# y-combinator

## How to make an anonimous recursive function

Start with a simple (named) recursive function:

```js
const isEven = n => n === 0 ? true : !isEven(n-1)

isEven(4) // true
isEven(5) // false
```

Place the self reference as a parameter:

```js
const almostIsEven = (self, n) => n === 0 ? true : !self(self, n-1)

almostIsEven(almostIsEven, 4) // true
almostIsEven(almostIsEven, 5) // false
```

We can easily come back to isEven from here:

```js
const almostIsEven = (self, n) => n === 0 ? true : !self(self, n-1)

const isEven = n => almostIsEven(almostIsEven, n)

isEven(4) // true
isEven(5) // false
```

We can introduce a combinator `m` to generalize that last operation:

```js
const almostIsEven = (self, n) => n === 0 ? true : !self(self, n-1)

const m = f => n => f(f, n)

const isEven = m(almostIsEven)

isEven(4) // true
isEven(5) // false
```

We can now remove all declarations to get an anonimous recursive function:

```js
const m = f => n => f(f, n)
m((self, n) => n === 0 ? true : !self(self, n-1))(4) // true
```

## Improvement

The previous result is fine but we had to modify the recursive function. It would be better if we could have something like:

```js
?(n => n === 0 ? true : !self(n-1))(4) // true
```

To get to that result we first need to curry our self reference, from `(a,b) => ...` to `a => b => ...` :

```js
const almostIsEven = self => n => n === 0 ? true : !self(self)(n-1)
```

Then our `m` operator needs to evolve too, and we can come back to isEven once again:

```js
const M = f => n => f(f)(n) // also simply  f => f(f)
const isEven = M(self => n => n === 0 ? true : !self(self)(n-1))
```

Now if `self` could take care of the `self(self)` part, we would have our result.

But that is literaly what `M` does. So we can introduce `Y`, similar to `M`:

```js
const M = f => n => f(f)(n)
const Y = f => n => f(Y(f))(n)
//                    ^
const isEven = Y(self => n => n === 0 ? true : !self(n-1))
```

That's good, but `Y` is now a named recursive function. Let's make it an anonimous recursive function by following the same steps as before:

```js
const almostY = self => f => n => f(self(self)(f))(n) // replaced Y by self(self)
const Y = M(almostY)
```

Or writen simply:

```js
const M = f => f(f)
const Y = M(m => f => a => f(m(m)(f))(a))

const isEven = Y(self => n => n === 0 ? true : !self(n-1))
```

`Y` is known as the Y-Combinator, it allows to transform any named recursive function into an anonymous function.

It is generaly know with the simpler form:

```js
const M = f => f(f)
const Y = f => M(m => a => f(m(m))(a))
```

Or without defining `M` first:

```js
const Y = f =>
    (m => a => f(m(m))(a))(
        m => a => f(m(m))(a)
    )
```

