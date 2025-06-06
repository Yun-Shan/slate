# Rendering

One of the best parts of Slate is that it's built with React, so it fits right into your existing application. It doesn't re-invent its own view layer that you have to learn. It tries to keep everything as React-y as possible.

To that end, Slate gives you control over the rendering behavior of your custom nodes and properties in your richtext domain.

You can define these behaviors by passing "render props" to the top-level `<Editable>` component.

For example if you wanted to render custom element components, you'd pass in the `renderElement` prop:

```jsx
import { createEditor } from 'slate'
import { Slate, Editable, withReact } from 'slate-react'

const MyEditor = () => {
  const [editor] = useState(() => withReact(createEditor()))
  const renderElement = useCallback(({ attributes, children, element }) => {
    switch (element.type) {
      case 'quote':
        return <blockquote {...attributes}>{children}</blockquote>
      case 'link':
        return (
          <a {...attributes} href={element.url}>
            {children}
          </a>
        )
      default:
        return <p {...attributes}>{children}</p>
    }
  }, [])

  return (
    <Slate editor={editor}>
      <Editable renderElement={renderElement} />
    </Slate>
  )
}
```

> 🤖 Be sure to mix in `props.attributes` and render `props.children` in your custom components! The attributes must be added to the top-level DOM element inside the component, as they are required for Slate's DOM helper functions to work. And the children are the "leaves" holding text content and inline elements.

You don't have to use simple HTML elements, you can use your own custom React components too:

```javascript
const renderElement = useCallback(props => {
  switch (props.element.type) {
    case 'quote':
      return <QuoteElement {...props} />
    case 'link':
      return <LinkElement {...props} />
    default:
      return <DefaultElement {...props} />
  }
}, [])
```

## Leaves

When text-level formatting is rendered, the characters are grouped into "leaves" of text that each contain the same formatting (marks) applied to them.

To customize the rendering of each leaf, you use a custom `renderLeaf` prop:

```jsx
const renderLeaf = useCallback(({ attributes, children, leaf }) => {
  return (
    <span
      {...attributes}
      style={{
        fontWeight: leaf.bold ? 'bold' : 'normal',
        fontStyle: leaf.italic ? 'italic' : 'normal',
      }}
    >
      {children}
    </span>
  )
}, [])
```

Notice though how we've handled it slightly differently than `renderElement`. Since text formatting tends to be fairly simple, we've opted to ditch the `switch` statement and just toggle on/off a few styles instead. \(But there's nothing preventing you from using custom components if you'd like!\)

> 🤖 As with the Element renderer, be sure to mix in `props.attributes` and render `props.children` in your leaf renderer! The attributes must be added to the top-level DOM element inside the component, as they are required for Slate's DOM helper functions to work. And the children are the actual text content of your document which Slate manages for you automatically.

When decorations split a single text node, the `renderLeaf` function will receive an additional `leafPosition` property. This object contains the `start` and `end` offsets of the leaf within the original text node, along with optional `isFirst` and `isLast` booleans. This `leafPosition` property is only added when a text node is actually split by decorations.

One disadvantage of text-level formatting is that you cannot guarantee that any given format is "contiguous"—meaning that it stays as a single leaf. This limitation with respect to leaves is similar to the DOM, where this is invalid:

```markup
<em>t<strong>e</em>x</strong>t
```

Because the elements in the above example do not properly close themselves they are invalid. Instead, you would write the above HTML as follows:

```markup
<em>t</em><strong><em>e</em>x</strong>t
```

If you happened to add another overlapping section of `<strike>` to that text, you might have to rearrange the closing tags yet again. Rendering leaves in Slate is similar—you can't guarantee that even though a word has one type of formatting applied to it that that leaf will be contiguous, because it depends on how it overlaps with other formatting.

Of course, this leaf stuff sounds pretty complex. But, you do not have to think about it much, as long as you use text-level formatting and element-level formatting for their intended purposes:

- Text properties are for **non-contiguous**, character-level formatting.
- Element properties are for **contiguous**, semantic elements in the document.

## Texts

While `renderLeaf` allows you to customize the rendering of individual leaves based on their formatting (marks and decorations), sometimes you need to customize the rendering for an entire text node, regardless of how decorations might split it into multiple leaves.

This is where the `renderText` prop comes in. It allows you to render a component that wraps all the leaves generated for a single `Text` node.

```jsx
const renderText = useCallback(({ attributes, children, text }) => {
  return (
    <span {...attributes} className="custom-text">
      {children}
      {/* Render anything you want here */}
    </span>
  )
}, [])

// In your editor component:
<Editable
  renderText={renderText}
  renderLeaf={renderLeaf}
/>
```

**When to use `renderLeaf` vs `renderText`:**

- **`renderLeaf`**: Use this when you need to apply styles or rendering logic based on the specific properties of each individual leaf (e.g., applying bold style if `leaf.bold` is true, or highlighting based on a decoration). This function might be called multiple times for a single text node if decorations split it. You can use the optional `leafPosition` prop (available when a text node is split) to conditionally render something based on the position of the leaf within the text node.

- **`renderText`**: Use this when you need to render something exactly once for a given text node, regardless of how many leaves it's split into. It's ideal for wrapping the entire text node's content or adding elements associated with the text node as a whole without worrying about duplication caused by decorations.

You can use both `renderText` and `renderLeaf` together. `renderLeaf` renders the individual marks and decorations within a text node (leaves), and `renderText` renders the container of those leaves.

## Decorations

Decorations are another type of text-level formatting. They are similar to regular old custom properties, except each one applies to a `Range` of the document instead of being associated with a given text node.

However, decorations are computed at **render-time** based on the content itself. This is helpful for dynamic formatting like syntax highlighting or search keywords, where changes to the content \(or some external data\) has the potential to change the formatting.

Decorations are different from Marks in that they are not stored on editor state.

## Toolbars, Menus, Overlays, and more!

In addition to controlling the rendering of nodes inside Slate, you can also retrieve the current editor context from inside other components using the `useSlate` hook.

That way other components can execute commands, query the editor state, or anything else.

A common use case for this is rendering a toolbar with formatting buttons that are highlighted based on the current selection:

```jsx
const MyEditor = () => {
  const [editor] = useState(() => withReact(createEditor()))
  return (
    <Slate editor={editor}>
      <Toolbar />
      <Editable />
    </Slate>
  )
}

const Toolbar = () => {
  const editor = useSlate()
  return (
    <div>
      <Button active={isBoldActive(editor)}>B</Button>
      <Button active={isItalicActive(editor)}>I</Button>
    </div>
  )
}
```

Because the `<Toolbar>` uses the `useSlate` hook to retrieve the context, it will re-render whenever the editor changes, so that the active state of the buttons stays in sync.

## Editor Styling

Custom styles can be applied to the editor itself by using the `style` prop on the `<Editable>` component.

```jsx
const MyEditor = () => {
  const [editor] = useState(() => withReact(createEditor()))
  return (
    <Slate editor={editor}>
      <Editable style={{ minHeight: '200px', backgroundColor: 'lime' }} />
    </Slate>
  )
}
```

It is also possible to apply custom styles with a stylesheet and `className`. However, Slate uses inline styles to provide some default styles for the editor. Because inline styles take precedence over stylesheets, styles you provide using stylesheets will not override the default styles. If you are trying to use a stylesheet and your rules are not taking effect, do one of the following:

- Provide your styles using the `style` prop instead of a stylesheet, which overrides the default inline styles.
- Pass the `disableDefaultStyles` prop to the `<Editable>` component.
- Use `!important` in your stylesheet declarations to make them override the inline styles.

## Performance

See [Improving Performance](../walkthroughs/09-performance.md) for ways to improve the rendering performance of the editor.
