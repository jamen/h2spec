# h2spec (WIP)
> A specification for proper `h()` calls.

## Abstract

Libraries such as [`hyperapp`](https://github.com/hyperapp), [`hyperscript`](https://github.com/hyperhype/hyperscript), [`choo`](https://github.com/yoshuawuyts/choo) and [`snabbdom`](https://github.com/snabbdom/snabbdom) often feature a notable discrepancy in their implementations of the `h()` function. The goal of `h2spec` is to revise and standardize the signature for `h()` functions under the name "h2" or "hyper2". These projects make the usage consistent no matter what view you're working with: DOM, virtual DOM, server rendering, terminal, canvas, etc. The spec itself is oriented around simplicity, usability, and ease of implementation.

Once the spec is more mature, several of these libraries will be forked and aligned with the spec. Patches will be sent upstream where it can be merged/rejected by the authors.  But, there will also be several "official" implementations before this as a proof-of-concept. Ones that get rejected may be maintained on their own.

## Specification

### `h(tag, data?, children?) -> node`

1. `tag`: a string denoting the desired tag name. Cannot contain extra data like classes or ids (e.g. `div.foo` or `div#bar`).
2. `data` (optional): a plain JavaScript object which maps attributes to values, otherwise `null` or `undefined`.
3. `children` (optional): an array of `node` objects (i.e. whatever the `h` function in question returns) and/or _primitive values_.

A primitive value is a boolean, number, string, `null`, or `undefined` (as specified [by ECMAScript](https://www.ecma-international.org/ecma-262/5.1/#sec-4.3.2)).

## Examples

```js
h('br')
h('div', { foo: 'bar' }, [ 'baz' ])
h('span', null, [ 'foo' ])
h('header', null, [
  h('h1', null, [ 'hello world' ]),
  h('nav', null, [
    h('a', { href: '#foo' }, [ 'foo' ]),
    h('a', { href: '#bar' }, [ 'bar' ]),
    h('a', { href: '#baz' }, [ 'baz' ])
  ])
])
```

## Projects

 - [`h2spec`](https://github.com/hyper2/h2spec) (this)
 - [`h2dom`](https://github.com/hyper2/h2dom) creates DOM nodes
 - [`h2ml`](https://github.com/hyper2/h2ml) creates HTML strings
 - [`h2array`](https://github.com/hyper2/h2json) creates array trees resembling `h()` calls

## FAQ

### Should the return types of `h`-libraries (DOM, VDOMs, strings, etc) cross into each other?

This spec isolates `h` functions' children to their own return type for a reason.
Do not mix return types outside of something experimental.
However, you can change your entire project tree to DOM, VDOM, and strings, because the elements created are compatible.

Elements are symbolized through combining `{ tag, data, children }`.
The `h(tag, data?, children?)` function is what ties the production of a DOM node, VDOM object, string, or anything, together with consistent usage under that simple combination.

While some of these return types do mix well (i.e. VDOMs + strings = SSR) most do not. Handling several content types in each creates needless complexity. This is exactly why `h(...)` should be very well defined in itself, so it is easy to swap out whole trees, because the libraries rely on the same `h` function usage.

### How would a VDOM or JSON abstraction of this look?

Since the elements are only tied together through `tag`, `data`, and `children`, you could just plop those into objects or arrays:

```js
{
  tag: 'div',
  data: { class: 'foo' },
  children: [
    { tag: 'div', data: null, children: [ 'foo' ] },
    { tag: 'span', data: null, children: [ 'bar' ] },
    // ...
  ]
}
```

or more compact

```js
[ 'div', { class: 'foo' }, [
  [ 'div', null, [ 'foo' ] ],
  [ 'span', null, [ 'bar' ] ]
]]
```

which could then map directly to `h()` calls:

```js
h('div', { class: 'foo' }, [
  h('div', null, [ 'foo' ]),
  h('span', null, [ 'bar' ])
])
```

This is possible because the spec limits trees to a single node type, so you can guarantee the `h(...)` function will be the same for every node.

### How strict is the spec?

Some conditions are critical, such as parameter precedence, but others are simple assertions that can be "reasonably ignored" for now to create smaller implementation sizes. For example, checking primitive types and array contents every time can be costly, so they can be assumed. But, an implementation should never support something outside of what the spec allows.

We may create fuzztesters to ensure compatbility as the spec gets more mature.
