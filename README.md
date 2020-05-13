# rubico
🏞 a shallow river in northeastern Italy, just south of Ravenna

a functional promise library

# Introduction
Asynchronous programming in JavaScript has evolved over the years

> In the beginning, there were [callbacks](http://callbackhell.com)
```javascript
function doAsyncThings(a, cb) {
  doAsyncThingA(a, function(errA, b) {
    if (errA) return cb(errA)
    doAsyncThingB(b, function(errB, c) {
      if (errB) return cb(errB)
      doAsyncThingC(c, function(errC, d) {
        if (errC) return cb(errC)
        doAsyncThingD(d, function(errD, e) {
          if (errD) return cb(errD)
          doAsyncThingE(e, function(errE, f) {
            if (errE) return cb(errE)
            cb(null, f)
          })
        })
      })
    })
  })
}
```

To stay within maximum line lengths, we created
[Promises](https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Promise.jsm/Promise)

> Then began the chains of `then`
```javascript
const doAsyncThings = a => doAsyncThingA(a)
  .then(b => doAsyncThingB(b))
  .then(c => doAsyncThingC(c))
  .then(d => doAsyncThingD(d))
  .then(e => doAsyncThingE(e))
```

This was fine until we started to miss variables

> [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), the latest in asynchrony
```javascript
const doAsyncThings = async a => {
  const b = await doAsyncThingA(a)
  const c = await doAsyncThingB(b)
  const d = await doAsyncThingC(c)
  const e = await doAsyncThingD(d)
  const f = await doAsyncThingE(e)
  return f
}
```

Some of us, however, did not miss variables

> maybe promises were on to something
```javascript
import { pipe } from 'rubico'

const doAsyncThings = pipe([
  doAsyncThingA,
  doAsyncThingB,
  doAsyncThingC,
  doAsyncThingD,
  doAsyncThingE,
])
```

Enter rubico, a functional (programming) promise library.

rubico resolves two promises:
1. simplify asynchronous programming in JavaScript
2. enable functional programming in JavaScript

programs written with rubico follow a [point-free style](https://en.wikipedia.org/wiki/Tacit_programming)

rubico works in server and browser JavaScript environments

# Installation
with deno;
```javascript
import {
  pipe, fork, assign, tap, tryCatch, switchCase,
  map, filter, reduce, transform,
  any, all, and, or, not,
  eq, gt, lt, gte, lte,
  get, pick, omit,
} from 'https://deno.land/x/rubico/mod.js'
```
with npm; `npm i rubico`

# Examples
The following examples compare algorithmically equivalent code between promise chains, async/await, and rubico

### Make a request
```javascript
// promise chains
fetch('https://jsonplaceholder.typicode.com/todos/1')
  .then(res => res.json())
  .then(console.log) // > {...}

// async/await
void (async () => {
  const res = await fetch('https://jsonplaceholder.typicode.com/todos/1')
  const data = await res.json()
  console.log(data) // > {...}
})()

// rubico
import { pipe } from 'rubico'

pipe([
  fetch,
  res => res.json(),
  console.log, // > {...}
])('https://jsonplaceholder.typicode.com/todos/1')
```

### Make multiple requests
```javascript
const todoIDs = [1, 2, 3, 4, 5]

// promise chains
Promise.resolve(todoIDs.filter(id => id <= 3))
  .then(filtered => Promise.all(filtered.map(
    id => `https://jsonplaceholder.typicode.com/todos/${id}`
  )))
  .then(urls => Promise.all(urls.map(fetch)))
  .then(responses => Promise.all(responses.map(res => res.json())))
  .then(data => data.map(x => console.log(x))) // > {...} {...} {...}

// async/await
void (async () => {
  const filtered = todoIDs.filter(id => id <= 3)
  const urls = await Promise.all(filtered.map(id => `https://jsonplaceholder.typicode.com/todos/${id}`))
  const responses = await Promise.all(urls.map(fetch))
  const data = await Promise.all(responses.map(res => res.json()))
  data.map(x => console.log(x)) // > {...} {...} {...}
})()

// rubico
import { pipe, map, filter } from 'rubico'

pipe([
  filter(id => id <= 3),
  map(id => `https://jsonplaceholder.typicode.com/todos/${id}`),
  map(fetch),
  map(res => res.json()),
  map(console.log), // > {...} {...} {...}
])(todoIDs)
```

# Documentation
rubico exports 23 functions

[pipe](https://github.com/richytong/rubico#pipe),
[fork](https://github.com/richytong/rubico#fork),
[assign](https://github.com/richytong/rubico#assign)
[tap](https://github.com/richytong/rubico#tap),
[tryCatch](https://github.com/richytong/rubico#tryCatch),
[switchCase](https://github.com/richytong/rubico#switchCase)

[map](https://github.com/richytong/rubico#map),
[filter](https://github.com/richytong/rubico#filter),
[reduce](https://github.com/richytong/rubico#reduce),
[transform](https://github.com/richytong/rubico#transform)

[any](https://github.com/richytong/rubico#any),
[all](https://github.com/richytong/rubico#all)
[and](https://github.com/richytong/rubico#and),
[or](https://github.com/richytong/rubico#or),
[not](https://github.com/richytong/rubico#not)

[eq](https://github.com/richytong/rubico#eq),
[gt](https://github.com/richytong/rubico#gt),
[lt](https://github.com/richytong/rubico#lt),
[gte](https://github.com/richytong/rubico#gte),
[lte](https://github.com/richytong/rubico#lte)

[get](https://github.com/richytong/rubico#get),
[pick](https://github.com/richytong/rubico#pick),
[omit](https://github.com/richytong/rubico#omit)

## pipe
chains sync or async functions from left to right
```javascript
y = pipe(functions)(x)
```
`functions` is an array of functions

`x` is anything

if `x` is a function, pipe chains `functions` from right to left,
see [transform](https://github.com/richytong/rubico#transform)

`y` is the output of running `x` through the chain of `functions`

if all functions of `functions` are synchronous, `y` is not a Promise

if any functions of `functions` are asynchronous, `y` is a Promise
```javascript
pipe([
  x => x + 'y',
  x => x + 'e',
  x => x + 'lmao',
])('a') // => 'ayelmao'

pipe([
  x => x + 'y',
  x => Promise.resolve(x + 'e'),
  async x => x + 'lmao',
])('a') // => Promise { 'ayelmao' }
```

## fork
parallelizes sync or async functions
```javascript
y = fork(functions)(x)
```
`functions` is either an array of functions or an object of functions

all functions of `functions` are run concurrently

`x` is anything

`y` assumes the shape of `functions`

`y` is the output of mapping `x` to each function of `functions`

if all functions of `functions` are synchronous, `y` is not a Promise

if any functions of `functions` are asynchronous, `y` is a Promise
```javascript
fork([
  x => 'o' + x + 'o',
  x => 'u' + x + 'u',
  x => 'x' + x + 'x',
])('w') // => ['owo', 'uwu', 'xwx']

fork([
  x => 'o' + x + 'o',
  x => Promise.resolve('u' + x + 'u'),
  async x => 'x' + x + 'x',
])('w') // => Promise { ['owo', 'uwu', 'xwx'] }

fork({
  a: x => x + 1,
  b: x => x + 2,
  c: x => x + 3,
})(0) // => { a: 1, b: 2, c: 3 }

fork({
  a: x => x + 1,
  b: x => Promise.resolve(x + 2),
  c: async x => x + 3,
})(0) // => Promise { { a: 1, b: 2, c: 3 } }
```

## assign
parallelizes sync or async functions, then merges result with input
```javascript
y = assign(functions)(x)
```
`functions` is an object of functions

all functions of `functions` are run concurrently

`x` is an object of anything

`y` is an object of anything

`y` is the output of mapping `x` to each function of `functions`, then merging `x` with that result

if all functions of `functions` are synchronous, `y` is not a Promise

if any functions of `functions` are asynchronous, `y` is a Promise

```javascript
assign({
  hello: x => 'hello ' + x.name,
  goodbye: x => 'goodbye ' + x.name,
})({ name: 'George' }) // => { hello: 'hello George', goodbye: 'goodbye George' }

assign({
  hello: x => 'hello ' + x.name,
  goodbye: async x => 'goodbye ' + x.name,
})({ name: 'George' }) // => Promise { { hello: 'hello George', goodbye: 'goodbye George' } }

assign({
  name: () => 'not George',
})({ name: 'George' }) // => { name: 'not George' }
```

## tap
calls a sync or async function with input, returning input
```javascript
y = tap(f)(x)
```
`x` is anything

`f` is a function that expects one argument `x`

if `f` is synchronous, `y` is `x`

if `f` is asynchronous, `y` is a Promise that resolves to `x`

if `x` is a function
```javascript
y = reduce(tap(f)(x))(z)
```
`reduce` is [reduce](https://github.com/richytong/rubico#reduce),

`z` is an iterable, asyncIterable, or object

`zi` is an element of `z`

`f` is a function that expects one argument `zi`

`y` is `reduce(x)(z)`
```javascript
tap(
  console.log, // > 'hey'
)('hey') // => 'hey'

tap(
  async x => console.log(x), // > 'hey'
)('hey') // => Promise { 'hey' }

reduce(
  tap(
    console.log, // > 1 2 3 4 5
  )((y, xi) => y + xi),
  0,
)([1, 2, 3, 4, 5]) // => 15
```

## tryCatch
tries a sync or async function with input, catches with another sync or async function
```javascript
y = tryCatch(f, g)(x)
```
`f` is a function that expects one argument `x`

`g` is a function that expects two arguments `err` and `x`

`err` is a value potentially thrown by `f(x)`

`x` is anything

if `f(x)` did not throw, `y` is `f(x)`

if `f(x)` threw, `y` is `g(err, x)`

if `f` and `g` are synchronous, `y` is not a Promise

if `f` is asynchronous, `y` is a Promise

if `f` is synchronous, `g` is asynchronous, and `f(x)` did not throw, `y` is not a Promise

if `f` is synchronous, `g` is asynchronous, and `f(x)` threw, `y` is a Promise
```javascript
tryCatch(
  x => x + 'yo',
  (e, x) => x + e.message,
)('a') // => 'ayo'

tryCatch(
  x => { throw new Error(x) },
  (e, x) => x + e.message,
)('a') // => 'aa'

tryCatch(
  async x => x + 'yo',
  (e, x) => x + e.message,
)('a') // => Promise { 'ayo' }

tryCatch(
  x => Promise.reject(new Error(x)),
  (e, x) => x + e.message,
)('a') // => Promise { 'aa' }
```

## switchCase
if1(x) ? do1(x) : if2(x) ? do2(x) : ... ifN(x) ? doN(x) : doDefault(x)
```javascript
y = switchCase(functions)(x)
```
`x` is anything

`functions` is an array of functions

given predicate functions `if1, if2, ..., ifN`;<br>
corresponding functions `do1, do2, ..., doN`;<br>
and default function `doDefault`;<br>
`functions` is the array of functions `[if1, do1, if2, do2, ..., ifN, doN, doDefault]`

switchCase evaluates supplied functions in series as `evaluated` and breaks early on a truthy predicate

if all functions of `evaluated` are synchronous, `y` is not a Promise

if any functions of `evaluated` are asynchronous, `y` is a Promise

```javascript
switchCase([
  x => x === 'a', x => x + 'yo',
  x => x === 'l', x => x + 'mao',
  () => '???',
])('a') // => 'ayo'

switchCase([
  x => x === 'a', x => x + 'yo',
  x => x === 'l', x => x + 'mao',
  () => '???',
])('b') // => '???'

switchCase([
  async x => x === 'a', x => x + 'yo',
  x => x === 'l', x => x + 'mao',
  () => '???',
])('a') // => Promise { 'ayo' }

switchCase([
  async x => x === 'a', x => x + 'yo',
  x => x === 'l', x => x + 'mao',
  () => '???',
])('b') // => Promise { '???' }

switchCase([
  x => x === 'a', x => x + 'yo',
  async x => x === 'l', async x => x + 'mao',
  async () => '???',
])('a') // => 'ayo'
```

## map
`y = map(f)(x)`

## filter
## reduce (WIP)
for each `zi` of `z`, `reduce` provides `x` with two arguments `y` and `zi`

`x(y, zi)` returns the `y` used with the next `zi`

`reduce` provides `x` with two arguments `y` and `zi`

`x` specifies the relationship between `y` and `zi`

if `x0` is provided, `y` is initially `x0`, else `y` assumes the first `zi` of `z`

## transform
## any
## all
## and
## or
## not
## eq
## gt
## lt
## gte
## lte
## get
## pick
## omit

# More Examples
### Sync vs async

### A webserver using map, transform, and https://deno.land/std/http/server.ts serve
```javascript
import { serve } from "https://deno.land/std/http/server.ts";
import { map, transform } from "https://deno.land/x/rubico/mod.js"
const s = serve({ port: 8001 });
console.log("http://localhost:8001/");
transform(null, map(req => {
  req.respond({ body: "Hello World\n" });
}))(s);
```

### A server with middleware
```javascript
import { serve } from 'https://deno.land/std/http/server.ts'
import {
  pipe, fork, assign, tap, tryCatch, switchCase,
  map, filter, reduce, transform,
  any, all, and, or, not,
  eq, gt, lt, gte, lte,
  get, pick, omit,
} from 'https://deno.land/x/rubico/mod.js'

const join = delim => x => x.join(delim)

const addServerTime = req => {
  req.serverTime = (new Date()).toJSON()
  return req
}

const traceRequest = pipe([
  fork([
    pipe([get('serverTime'), x => '[' + x + ']']),
    get('method'),
    get('url'),
  ]),
  join(' '),
  console.log,
])

const respondWithHelloWorld = req => {
  req.respond({ body: 'Hello World\n' })
}

const respondWithServerTime = req => {
  req.respond({ body: `The server time is ${req.serverTime}` })
}

const respondWithNotFound = req => {
  req.respond({ body: 'Not Found\n' })
}

const route = switchCase([
  eq('/', get('url')), respondWithHelloWorld,
  eq('/time', get('url')), respondWithServerTime,
  respondWithNotFound,
])

const onRequest = pipe([
  addServerTime,
  tap(traceRequest),
  route,
])

const s = serve({ port: 8001 })
console.log('http://localhost:8001/')
transform(null, map(onRequest))(s)
```
