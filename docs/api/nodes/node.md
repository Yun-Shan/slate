# Node API

- [Static methods](node.md#static-methods)
  - [Retrieval methods](node.md#retrieval-methods)
  - [Text methods](node.md#text-methods)
  - [Check methods](node.md#check-methods)

## Static methods

### Retrieval methods

#### `Node.ancestor(root: Node, path: Path) => Ancestor`

Get the node at a specific `path`, asserting that it is an ancestor node. If the specified node is not an ancestor node, throw an error.

#### `Node.ancestors(root: Node, path: Path, options?) => Generator<NodeEntry<Ancestor>>`

Return a generator of all the ancestor nodes above a specific path. By default, the order is top-down, from highest to lowest ancestor in the tree, but you can pass the `reverse: true` option to go bottom-up.

Options: `{reverse?: boolean}`

#### `Node.child(root: Node, index: number) => Descendant`

Get the child of a node at the specified `index`.

#### `Node.children(root: Node, path: Path, options?) => Generator<NodeEntry<Descendant>>`

Iterate over the children of a node at a specific path.

Options: `{reverse?: boolean}`

#### `Node.common(root: Node, path: Path, another: Path) => NodeEntry`

Get an entry for the common ancestor node of two paths. It might be a Text node, an Element, or the Editor itself.

For the common block ancestor, see [Editor Selection](https://docs.slatejs.org/concepts/03-locations#selection)

#### `Node.descendant(root: Node, path: Path) => Descendant`

Get the node at a specific path, asserting that it's a descendant node.

#### `Node.descendants(root: Node, options?) => Generator<NodeEntry<Descendant>>`

Return a generator of all the descendant node entries inside a root node. Each iteration will return a `NodeEntry` tuple consisting of `[Node, Path]`.

Options: `{from?: Path, to?: Path, reverse?: boolean, pass?: (node: NodeEntry => boolean)}`

#### `Node.elements(root: Node, options?) => Generator<ElementEntry>`

Return a generator of all the element nodes inside a root node. Each iteration will return an `ElementEntry` tuple consisting of `[Element, Path]`. If the root node is an element, it will be included in the iteration as well.

Options: `{from?: Path, to?: Path, reverse?: boolean, pass?: (node: NodeEntry => boolean)}`

#### `Node.extractProps(node: Node) => NodeProps`

Extract all properties from a Node except for its content-related fields (`children` for Element nodes and `text` for Text nodes).

```typescript
// For an Element node
const element = {
  type: 'paragraph',
  align: 'center',
  children: [{ text: 'Try it out for yourself!' }],
}
const props = Node.extractProps(element)
// Returns: { type: 'paragraph', align: "center" }

// For a Text node
const text = { text: 'Hello', bold: true }
const props = Node.extractProps(text)
// Returns: { bold: true }
```

#### `Node.first(root: Node, path: Path) => NodeEntry`

Get the first node entry in a root node from a `path`.

#### `Node.fragment(root: Node, range: Range) => Descendant[]`

Get the sliced fragment represented by the `range`.

#### `Node.get(root: Node, path: Path) => Node`

Get the descendant node referred to by a specific `path`. If the path is an empty array, get the root node itself.

#### `Node.getIf(root: Node, path: Path) => Node | undefined`

Get a descendant node at a specific path, returning `undefined` if the node does not exist. This is a safer alternative to `Node.get()` as it won't throw an error if the path is invalid.

```typescript
const node = Node.getIf(root, [0, 1])
if (node) {
  // node exists at path [0, 1]
} else {
  // no node exists at path [0, 1]
}
```

#### `Node.last(root: Node, path: Path) => NodeEntry`

Get the last node entry in a root node at a specific `path`.

#### `Node.leaf(root: Node, path: Path) => Text`

Get the node at a specific `path`, ensuring it's a leaf text node. If the node is not a leaf text node, throw an error.

#### `Node.levels(root: Node, path: Path, options?) => Generator<NodeEntry>`

Return a generator of the nodes in a branch of the tree, from a specific `path`. By default, the order is top-down, from the highest to the lowest node in the tree, but you can pass the `reverse: true` option to go bottom-up.

Options: `{reverse?: boolean}`

#### `Node.nodes(root: Node, options?) => Generator<NodeEntry>`

Return a generator of all the node entries of a root node. Each entry is returned as a `[Node, Path]` tuple, with the path referring to the node's position inside the root node.

Options: `{from?: Path, to?: Path, reverse?: boolean, pass?: (node: NodeEntry => boolean)}`

#### `Node.parent(root: Node, path: Path) => Ancestor`

Get the parent of a node at a specific `path`.

### Text methods

Methods related to Text.

#### `Node.string(root: Node) => string`

Get the concatenated text string of a node's content. Note that this will not include spaces or line breaks between block nodes. This is not intended as a user-facing string, but as a string for performing offset-related computations for a node.

#### `Node.texts(root: Node, options?) => Generator<NodeEntry<Text>>`

Return a generator of all leaf text nodes in a root node.

Options: `{from?: Path, to?: Path, reverse?: boolean, pass?: (node: NodeEntry => boolean)}`

### Check methods

Methods used to check some attribute of a Node.

#### `Node.has(root: Node, path: Path) => boolean`

Check if a descendant node exists at a specific `path`.

#### `Node.isNode(value: any) => value is Node`

Check if a `value` implements the `Node` interface.

#### `Node.isNodeList(value: any) => value is Node[]`

Check if a `value` is a list of `Node` objects.

#### `Node.matches(root: Node, props: Partial<Node>) => boolean`

Check if a node matches a set of `props`.
