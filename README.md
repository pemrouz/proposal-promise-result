# ECMAScript proposal: `Promise.result`

## Status

This proposal is at stage 0 of [the TC39 process](https://tc39.github.io/process-document/).

## Motivation

Recursively unwrapping is very useful when working with Promises. However, there are some scenarios where this is undesirable. 

The main motivation arises for platform, framework or library APIs that deal with arbitrary input. Module namespace objects ([Issue 1](https://github.com/tc39/proposal-dynamic-import/issues/47), [Issue 2](https://github.com/tc39/proposal-dynamic-import/issues/48)) are affected by this, if you export a `then` function. This has been overwhemingly [problematic](https://twitter.com/Jhnnns/status/1100074123118735360) and [suprising](https://twitter.com/DasSurma/status/1101461224317956101) for users, in most cases tripping over it [by accident](https://github.com/ramda/ramda/issues/2751). This is the first major case: fetching values from storage using IndexedDB or KV storage, is not affected since you can't store functions in the first place. However as mentioned in several places it's a generic problem [that needs a generic solution](https://github.com/tc39/proposal-dynamic-import/issues/47#issuecomment-322230094). This problem is not limited to platform boundaries either - for example if you have a web server that uses an async function to process a request/response, you could not intentionally resolve to an object with `then`, or may trip over this unintentionally if you don't add extra code to check for it. 

```js
// fine
async function processor(req, res){
  return { foo, bar }
}

// not fine
async function processor(req, res){
  return { foo, bar, then } 
}
```

You would have to box the value before returning. The framework would have to know about that specific box too. 

This proposal reflects the current consensus of the committee and approach of implementors working around this issue today: if you aren't expecting a thenable, then wrap it. The motivation behind having a standard wrapper is to provide the ecosystem (primarily platform, but also framework and library authors) with a clear mechanism to avoid these unintended consequences in the future. 

## Proposed solution

`Promise.result` is a way to explicitly opt-in to boxing a value. It does not break or change the existing thenable-assimilation behaviour. The boxed value can then propagate through the async-await machinery. The wrapper is thrown away at the end. 

## Example

```js
async function foo(){
  return { value: 'foo', then(resolve){ resolve('bar') } }
}

await foo() // bar
```

```js
async function foo(){
  return Promise.result({ value: 'foo', then(resolve){ resolve('bar') } })
}

await foo() // { value: 'foo', then(){} }
```

## FAQ

#### Why not `Symbol.thenable`?

[`Symbol.thenable`](https://github.com/devsnek/proposal-symbol-thenable/blob/master/README.md) ([Meeting Notes](https://tc39.github.io/tc39-notes/2018-05_may-24.html#symbolthenable-for-stage-1-or-2)) had a number of issues raised which were specific to that proposal e.g. `Symbol.thenable = false` to mark something as unthenable feeling backwards. A different design to using a Symbol was suggested.

#### Why is this different to other protocol hooks like `valueOf` or `toString`?

`valueOf` or `toString` do not have any recursive behaviour. If they did, for example perhaps if `toString` crawled up the prototype chain, then most likely we would need a way to explicitly draw the line there too.

#### Does this add more complexity?

This is the current consensus of the committee and approach of implementors working around this issue: if you aren't expecting a thenable, then wrap it. So no new complexity is added. The complexity is decreased overall as there is only one wrapper for this in the ecosystem as opposed to everybody using different ones and having to check for wrappers. 

#### Are we breaking web compatibility?

There are some who believe confusing module namespace objects as user-land thenables are a [serious design bug](https://twitter.com/awbjs/status/1100174916983214080) to the extent that they are strongly opposed to it progressing from Stage 3 to 4 until it's fixed. 

At a minimum this proposal would solve the problem for future APIs. The idea for dynamic import would be that the module namespace could be (or should have been already!) internally wrapped with `Promise.result` so when it's awaited by users it's resolved to the correct value. If this receives Stage 1, then the possiblity of fixing dynamic imports can be investigated for the next meeting.

## TC39 meeting notes

- TODO

## Specification

- [Ecmarkup source]()
- [HTML version]()

## Implementations

- none yet
