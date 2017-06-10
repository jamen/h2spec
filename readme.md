
## Spec

### `h(name, data?, children?) -> node`

1. `name` must be type of `string` and **cannot** carry extra data like classes or ids (i.e. `div.foo`/`div#bar`)
2. If 3rd param is `undefined` and 2nd param is true for steps 2 or 3, it becomes `data` or `children` repsectively
3. `data` can be an object (excluding arrays) only if the `node` type is not also an object, otherwise `null`
4. `children` can only be _empty_, a _primitive value_, the type of `node`, or an array containing any of those types
 
- _empty_ is a "no value node" type returned by `h`.  e.g. an empty node object, empty array, or empty string
- _primitive value_ is `undefined`, `null`, Boolean, Number, or String (defined [by ECMAScript](https://www.ecma-international.org/ecma-262/5.1/#sec-4.3.2))

Some conditions are critical such as the parameter's percedence, but others are simple assertions that can be "reasonably ignored" to create smaller implementation sizes.  For example, checking primitive types and array contents every time can be costly, so they can be assumed.  But, an implemetation should never support something outside of what the spec allows

## Projects

 - [`h2ml`](https://github.com/hyper2/h2ml) creates an HTML string
 - [`h2json`](https://github.com/hyper2/h2json) creates a compact tree of arrays for JSON

## FAQ

### Should the return types of `h`-libraries (DOM, VDOMs, strings, etc) cross into each other?

This spec isolates `h` functions' children to their own return type for a reason.
Do not mix return types outside of something experimental.  However, feel free to change
your entire project tree between DOM, VDOM, and strings, since how the elements are created is
compatible

Elements are symbolized through combining `{ name, data, children }`.
The `h(name, data?, children?)` function is what ties the production of a DOM node, VDOM object, string,
or anything, together with nice usage under that simple combination

While some of these return types do mix well, like VDOMs + strings = SSR, most do not. Handling several content types 
in each creates needless complexity.  This is exactly why `h(...)` should be very well defined in itself,
so it is easy to swap out whole trees, because the libraries rely on the same `h` function usage. 

### How would a VDOM or JSON abstraction of this look?

Since the elements are only tied together through `name`, `data`, and `children`, you could
just plop those into objects or arrays:

```js
{ name: 'div', data: { class: 'foo' }, children: [
    { name: 'div', data: null, children: 'foo' },
    { name: 'span', data: null, children: 'bar' },
    // ...
] }
```

Or more compact

```js
['div', {class: 'foo'}, [
   ['div', 'foo'],
   ['span', 'bar']
]]
```

Which could map directly to `h()` calls:

```js
h('div', {class: 'foo'}, [
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


