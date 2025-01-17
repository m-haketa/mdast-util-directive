# mdast-util-directive

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

[mdast][] extensions to parse and serialize [generic directives proposal][prop]
(`:cite[smith04]`, `::youtube[Video of a cat in a box]{v=01ab2cd3efg}`, and
such).

## Contents

*   [What is this?](#what-is-this)
*   [When to use this](#when-to-use-this)
*   [Install](#install)
*   [Use](#use)
*   [API](#api)
    *   [`directiveFromMarkdown`](#directivefrommarkdown)
    *   [`directiveToMarkdown`](#directivetomarkdown)
    *   [`ContainerDirective`](#containerdirective)
    *   [`Directive`](#directive)
    *   [`LeafDirective`](#leafdirective)
    *   [`TextDirective`](#textdirective)
*   [HTML](#html)
*   [Syntax](#syntax)
*   [Syntax tree](#syntax-tree)
    *   [Nodes](#nodes)
    *   [Mixin](#mixin)
*   [Types](#types)
*   [Compatibility](#compatibility)
*   [Related](#related)
*   [Contribute](#contribute)
*   [License](#license)

## What is this?

This package contains two extensions that add support for directive syntax in
markdown to [mdast][].
These extensions plug into
[`mdast-util-from-markdown`][mdast-util-from-markdown] (to support parsing
directives in markdown into a syntax tree) and
[`mdast-util-to-markdown`][mdast-util-to-markdown] (to support serializing
directives in syntax trees to markdown).

## When to use this

Directives are one of the four ways to extend markdown: an arbitrary extension
syntax (see [Extending markdown][extending-markdown] in micromark’s docs for
the alternatives and more info).
This mechanism works well when you control the content: who authors it, what
tools handle it, and where it’s displayed.
When authors can read a guide on how to embed a tweet but are not expected to
know the ins and outs of HTML or JavaScript.
Directives don’t work well if you don’t know who authors content, what tools
handle it, and where it ends up.
Example use cases are a docs website for a project or product, or blogging tools
and static site generators.

You can use these extensions when you are working with
`mdast-util-from-markdown` and `mdast-util-to-markdown` already.

When working with `mdast-util-from-markdown`, you must combine this package
with [`micromark-extension-directive`][extension].

When you don’t need a syntax tree, you can use [`micromark`][micromark]
directly with `micromark-extension-directive`.

All these packages are used [`remark-directive`][remark-directive], which
focusses on making it easier to transform content by abstracting these
internals away.

This package only handles the syntax tree.
For example, it does not handle how markdown is turned to HTML.
You can use this with some more code to match your specific needs, to allow for
anything from callouts, citations, styled blocks, forms, embeds, spoilers, etc.
[Traverse the tree][traversal] to change directives to whatever you please.

## Install

This package is [ESM only][esm].
In Node.js (version 14.14+ and 16.0+), install with [npm][]:

```sh
npm install mdast-util-directive
```

In Deno with [`esm.sh`][esmsh]:

```js
import {directiveFromMarkdown, directiveToMarkdown} from 'https://esm.sh/mdast-util-directive@2'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import {directiveFromMarkdown, directiveToMarkdown} from 'https://esm.sh/mdast-util-directive@2?bundle'
</script>
```

## Use

Say our document `example.md` contains:

```markdown
A lovely language know as :abbr[HTML]{title="HyperText Markup Language"}.
```

…and our module `example.js` looks as follows:

```js
import fs from 'node:fs/promises'
import {fromMarkdown} from 'mdast-util-from-markdown'
import {toMarkdown} from 'mdast-util-to-markdown'
import {directive} from 'micromark-extension-directive'
import {directiveFromMarkdown, directiveToMarkdown} from 'mdast-util-directive'

const doc = await fs.readFile('example.md')

const tree = fromMarkdown(doc, {
  extensions: [directive()],
  mdastExtensions: [directiveFromMarkdown]
})

console.log(tree)

const out = toMarkdown(tree, {extensions: [directiveToMarkdown]})

console.log(out)
```

…now running `node example.js` yields (positional info removed for brevity):

```js
{
  type: 'root',
  children: [
    {
      type: 'paragraph',
      children: [
        {type: 'text', value: 'A lovely language know as '},
        {
          type: 'textDirective',
          name: 'abbr',
          attributes: {title: 'HyperText Markup Language'},
          children: [{type: 'text', value: 'HTML'}]
        },
        {type: 'text', value: '.'}
      ]
    }
  ]
}
```

```markdown
A lovely language know as :abbr[HTML]{title="HyperText Markup Language"}.
```

## API

This package exports the identifiers
[`directiveFromMarkdown`][api-directivefrommarkdown] and
[`directiveToMarkdown`][api-directivetomarkdown].
There is no default export.

### `directiveFromMarkdown`

Extension for [`mdast-util-from-markdown`][mdast-util-from-markdown] to enable
directives ([`FromMarkdownExtension`][frommarkdownextension]).

### `directiveToMarkdown`

Extension for [`mdast-util-to-markdown`][mdast-util-to-markdown] to enable
directives ([`ToMarkdownExtension`][tomarkdownextension]).

There are no options, but passing [`options.quote`][quote] to
`mdast-util-to-markdown` is honored for attributes.

### `ContainerDirective`

Directive in flow content (such as in the root document, or block quotes),
which contains further flow content (TypeScript type).

###### Type

```ts
import type {BlockContent, DefinitionContent, Parent} from 'mdast'

interface ContainerDirective extends Parent {
  type: 'containerDirective'
  name: string
  attributes?: Record<string, string | null | undefined> | null | undefined
  children: Array<BlockContent | DefinitionContent>
}
```

### `Directive`

The different directive nodes (TypeScript type).

###### Type

```ts
type Directive = ContainerDirective | LeafDirective | TextDirective
```

### `LeafDirective`

Directive in flow content (such as in the root document, or block quotes),
which contains nothing (TypeScript type).

###### Type

```ts
import type {PhrasingContent, Parent} from 'mdast'

interface LeafDirective extends Parent {
  type: 'leafDirective'
  name: string
  attributes?: Record<string, string | null | undefined> | null | undefined
  children: Array<PhrasingContent>
}
```

### `TextDirective`

Directive in phrasing content (such as in paragraphs, headings) (TypeScript
type).

###### Type

```ts
import type {PhrasingContent, Parent} from 'mdast'

interface TextDirective extends Parent {
  type: 'textDirective'
  name: string
  attributes?: Record<string, string | null | undefined> | null | undefined
  children: Array<PhrasingContent>
}
```

## HTML

This utility does not handle how markdown is turned to HTML.
You can use this with some more code to match your specific needs, to allow for
anything from callouts, citations, styled blocks, forms, embeds, spoilers, etc.
[Traverse the tree][traversal] to change directives to whatever you please.

## Syntax

See [Syntax in `micromark-extension-directive`][syntax].

## Syntax tree

The following interfaces are added to **[mdast][]** by this utility.

### Nodes

#### `TextDirective`

```idl
interface TextDirective <: Parent {
  type: 'textDirective'
  children: [PhrasingContent]
}

TextDirective includes Directive
```

**TextDirective** (**[Parent][dfn-parent]**) is a directive.
It can be used where **[phrasing][dfn-phrasing-content]** content is expected.
Its content model is also **[phrasing][dfn-phrasing-content]** content.
It includes the mixin **[Directive][dfn-mxn-directive]**.

For example, the following Markdown:

```markdown
:name[Label]{#x.y.z key=value}
```

Yields:

```js
{
  type: 'textDirective',
  name: 'name',
  attributes: {id: 'x', class: 'y z', key: 'value'},
  children: [{type: 'text', value: 'Label'}]
}
```

#### `LeafDirective`

```idl
interface LeafDirective <: Parent {
  type: 'leafDirective'
  children: [PhrasingContent]
}

LeafDirective includes Directive
```

**LeafDirective** (**[Parent][dfn-parent]**) is a directive.
It can be used where **[flow][dfn-flow-content]** content is expected.
Its content model is **[phrasing][dfn-phrasing-content]** content.
It includes the mixin **[Directive][dfn-mxn-directive]**.

For example, the following Markdown:

```markdown
::youtube[Label]{v=123}
```

Yields:

```js
{
  type: 'leafDirective',
  name: 'youtube',
  attributes: {v: '123'},
  children: [{type: 'text', value: 'Label'}]
}
```

#### `ContainerDirective`

```idl
interface ContainerDirective <: Parent {
  type: 'containerDirective'
  children: [FlowContent]
}

ContainerDirective includes Directive
```

**ContainerDirective** (**[Parent][dfn-parent]**) is a directive.
It can be used where **[flow][dfn-flow-content]** content is expected.
Its content model is also **[flow][dfn-flow-content]** content.
It includes the mixin **[Directive][dfn-mxn-directive]**.

The phrasing in the label is, when available, added as a paragraph with a
`directiveLabel: true` field, as the head of its content.

For example, the following Markdown:

```markdown
:::spoiler[Open at your own peril]
He dies.
:::
```

Yields:

```js
{
  type: 'containerDirective',
  name: 'spoiler',
  attributes: {},
  children: [
    {
      type: 'paragraph',
      data: {directiveLabel: true},
      children: [{type: 'text', value: 'Open at your own peril'}]
    },
    {
      type: 'paragraph',
      children: [{type: 'text', value: 'He dies.'}]
    }
  ]
}
```

### Mixin

#### `Directive`

```idl
interface mixin Directive {
  name: string
  attributes: Attributes?
}

interface Attributes {}
typedef string AttributeName
typedef string AttributeValue
```

**Directive** represents something defined by an extension.

The `name` field must be present and represents an identifier of an extension.

The `attributes` field represents information associated with the node.
The value of the `attributes` field implements the **Attributes** interface.

In the **Attributes** interface, every field must be an `AttributeName` and
every value an `AttributeValue`.
The fields and values can be anything: there are no semantics (such as by HTML
or hast).

> In JSON, the value `null` must be treated as if the attribute was not
> included.
> In JavaScript, both `null` and `undefined` must be similarly ignored.

## Types

This package is fully typed with [TypeScript][].
It exports the additional types [`ContainerDirective`][api-containerdirective],
[`Directive`][api-directive], [`LeafDirective`][api-leafdirective], and
[`TextDirective`][api-textdirective].

It also registers the node types with `@types/mdast`.
If you’re working with the syntax tree, make sure to import this utility
somewhere in your types, as that registers the new node types in the tree.

```js
/**
 * @typedef {import('mdast-util-directive')}
 */

import {visit} from 'unist-util-visit'

/** @type {import('mdast').Root} */
const tree = getMdastNodeSomeHow()

visit(tree, (node) => {
  // `node` can now be one of the nodes for directives.
})
```

## Compatibility

Projects maintained by the unified collective are compatible with all maintained
versions of Node.js.
As of now, that is Node.js 14.14+ and 16.0+.
Our projects sometimes work with older versions, but this is not guaranteed.

This plugin works with `mdast-util-from-markdown` version 1+ and
`mdast-util-to-markdown` version 1+.

## Related

*   [`remarkjs/remark-directive`][remark-directive]
    — remark plugin to support generic directives
*   [`micromark/micromark-extension-directive`][extension]
    — micromark extension to parse directives

## Contribute

See [`contributing.md`][contributing] in [`syntax-tree/.github`][health] for
ways to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## License

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/syntax-tree/mdast-util-directive/workflows/main/badge.svg

[build]: https://github.com/syntax-tree/mdast-util-directive/actions

[coverage-badge]: https://img.shields.io/codecov/c/github/syntax-tree/mdast-util-directive.svg

[coverage]: https://codecov.io/github/syntax-tree/mdast-util-directive

[downloads-badge]: https://img.shields.io/npm/dm/mdast-util-directive.svg

[downloads]: https://www.npmjs.com/package/mdast-util-directive

[size-badge]: https://img.shields.io/bundlephobia/minzip/mdast-util-directive.svg

[size]: https://bundlephobia.com/result?p=mdast-util-directive

[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg

[backers-badge]: https://opencollective.com/unified/backers/badge.svg

[collective]: https://opencollective.com/unified

[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg

[chat]: https://github.com/syntax-tree/unist/discussions

[npm]: https://docs.npmjs.com/cli/install

[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c

[esmsh]: https://esm.sh

[typescript]: https://www.typescriptlang.org

[license]: license

[author]: https://wooorm.com

[health]: https://github.com/syntax-tree/.github

[contributing]: https://github.com/syntax-tree/.github/blob/main/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/main/support.md

[coc]: https://github.com/syntax-tree/.github/blob/main/code-of-conduct.md

[mdast]: https://github.com/syntax-tree/mdast

[mdast-util-from-markdown]: https://github.com/syntax-tree/mdast-util-from-markdown

[mdast-util-to-markdown]: https://github.com/syntax-tree/mdast-util-to-markdown

[quote]: https://github.com/syntax-tree/mdast-util-to-markdown#optionsquote

[micromark]: https://github.com/micromark/micromark

[extension]: https://github.com/micromark/micromark-extension-directive

[syntax]: https://github.com/micromark/micromark-extension-directive#syntax

[remark-directive]: https://github.com/remarkjs/remark-directive

[extending-markdown]: https://github.com/micromark/micromark#extending-markdown

[prop]: https://talk.commonmark.org/t/generic-directives-plugins-syntax/444

[traversal]: https://unifiedjs.com/learn/recipe/tree-traversal/

[dfn-parent]: https://github.com/syntax-tree/mdast#parent

[dfn-flow-content]: https://github.com/syntax-tree/mdast#flowcontent

[dfn-phrasing-content]: https://github.com/syntax-tree/mdast#phrasingcontent

[frommarkdownextension]: https://github.com/syntax-tree/mdast-util-from-markdown#extension

[tomarkdownextension]: https://github.com/syntax-tree/mdast-util-to-markdown#options

[api-directivefrommarkdown]: #directivefrommarkdown

[api-directivetomarkdown]: #directivetomarkdown

[api-containerdirective]: #containerdirective

[api-directive]: #directive

[api-leafdirective]: #leafdirective

[api-textdirective]: #textdirective

[dfn-mxn-directive]: #directive-1
