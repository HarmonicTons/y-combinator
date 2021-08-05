# y-combinator

## How to make an anonymous recursive function

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

Now if `self` could take care of the `self(self)` part when called, we would have our result.

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

## Lambda notation

In lambda notation a function that takes x and return t `x => t` is written like this: `λx.t`.

So for instance the M operator `x => x(x)` becomes `λx.x x` (parenthesis are removed when possible). In Lambda calculus M is called ω.

The Y operator second part `m => a => f(m(m))(a)` is equivalent to `m => f(m(m))` which becomes `λx.f(x x)`.

And so the Y operator `f => M(m => a => f(m(m))(a))` becomes `λf.(λx.x x)(λx.f(x x))`.

We can also define Ω as `Ω = ω ω` which is equivalent to `M(M)`. But as `M = f => f(f)` that means that `M(M) = M(M)` or `Ω = Ω`. Ω is only equal to itself and nothing else. It is the quintescence of the recursivity, the infinite loop. 

## Fibonacci 

A simple definition for fibonacci can be:

```js
const fib = (i) => {
    if (i <= 1) {
        return 1
    } else {
        return fib(i-1) + fib(i-2)
    }
}
```

In lambda calculus it would be: `Y( λf.λa. if(leg(a)(1)) (1) (plus (f(pred(a))) (f(pred(pred(a)))) ) )`

Or back in JS:

```js
const B = f => g => a => f(g(a))
const succ = n => f => B(f)(n(f))
const add = n => k => n(succ)(k)
const cond = p => a => b => p(a)(b)
const n0 = f => a => a
const n1 = f => a => f(a)
const n2 = add(n1)(n1)
const n3 = add(n1)(n2)
const n4 = add(n1)(n3)
const n5 = add(n1)(n4)
const pair = a => b => f => f(a)(b)
const K = a => b => a
const fst = p => p(K)
const I = a => a
const snd = p => p(K(I))
const phi = p => pair(snd(p))(succ(snd(p)))
const pred = n => fst(n(phi)(pair(n0)(n0)))
const F = a => b => b 
const T = a => b => a
const is0 = n => n(K(F))(T)
const sub = n => k => k(pred)(n)
const leq = n => k => is0(sub(n)(k))
const M = f => f(f)
const Y = M(m => f => a => f(m(m)(f))(a))

const almostFib = f => a => (cond (leq(a)(n1)) (g => n1) (g => add (g(pred(a))) (g(pred(pred(a))))))(f)
const fib = Y(almostFib)

fib(n5)(n => n + 1)(0) // 8
```
