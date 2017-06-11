
## Spec

### `h(tag, data?, children?) -> node`

1. `tag` must be a string and can not contain extra data like classes or ids (e.g. `div.foo` or `div#bar`)
2. If the 3rd parameter is not defined
    1. And the 2nd parameter passes step 3, then the 2nd becomes `data`
    2. And the 2nd parameter passes step 4, then the 2nd becomes `children`
3. `data` must only be a default javascript object unless `node` is also an object, otherwise it must be `null`
4. `children` must only be a `node`, _primitive value_, or an array of either

- _primitive value_ is Boolean, Number, String, `null`, or `undefined` (specified [by ECMAScript](https://www.ecma-international.org/ecma-262/5.1/#sec-4.3.2))

Some conditions are criticali, such as the parameter's precedence, but others are simple assertions that can be "reasonably ignored" to create smaller implementation sizes. For example, checking primitive types and array contents every time can be costly, so they can be assumed. But, an implementation should never support something outside of what the spec allows.

## Projects

 - [`h2ml`](https://github.com/hyper2/h2ml) creates an HTML string
 - [`h2json`](https://github.com/hyper2/h2json) creates a compact tree of arrays for JSON

## FAQ

### Should the return types of `h`-libraries (DOM, VDOMs, strings, etc) cross into each other?

This spec isolates `h` functions' children to their own return type for a reason.
Do not mix return types outside of something experimental. However, feel free to change your entire project tree between DOM, VDOM, and strings, since how the elements are created is compatible

Elements are symbolized through combining `{ tag, data, children }`.
The `h(tag, data?, children?)` function is what ties the production of a DOM node, VDOM object, string, or anything, together with nice usage under that simple combination

While some of these return types do mix well, like VDOMs + strings = SSR, most do not. Handling several content types in each creates needless complexity.
This is exactly why `h(...)` should be very well defined in itself, so it is easy to swap out whole trees, because the libraries rely on the same `h` function usage.

### How would a VDOM or JSON abstraction of this look?

Since the elements are only tied together through `tag`, `data`, and `children`, you could just plop those into objects or arrays:

```js
{
  tag: 'div',
  data: { class: 'foo' },
  children: [
    { tag: 'div', data: null, children: 'foo' },
    { tag: 'span', data: null, children: 'bar' },
    // ...
  ]
}
```

Or more compact

```js
['div', { class: 'foo' }, [
  ['div', 'foo'],
  ['span', 'bar']
]]
```

Which could map directly to `h()` calls:

```js
h('div', { class: 'foo' }, [
  h('div', 'foo'),
  h('span', 'bar')
])
```

This is possible because the spec limits trees to a single node type, so you can guarantee the `h(...)` function will be the same for every node.

### Example of possible uses?

```js
h('div', { ...data }, [ ...children|empty ])
h('div', { ...data }, child|empty)
h('div', null, children)
h('div', children)
h('div', { ...data })
h('div', null)
h('div')
```
