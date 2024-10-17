## Flex Container Properties

| Property | Syntax | Description |
|----------|--------|-------------|
| display | `display: flex;` | Defines a flex container |
| flex-direction | `flex-direction: row \| row-reverse \| column \| column-reverse;` | Sets the direction of the main axis |
| flex-wrap | `flex-wrap: nowrap \| wrap \| wrap-reverse;` | Controls whether items wrap to new lines |
| flex-flow | `flex-flow: <flex-direction> <flex-wrap>;` | Shorthand for flex-direction and flex-wrap |
| justify-content | `justify-content: flex-start \| flex-end \| center \| space-between \| space-around \| space-evenly;` | Aligns items along the main axis |
| align-items | `align-items: stretch \| flex-start \| flex-end \| center \| baseline;` | Aligns items along the cross axis |
| align-content | `align-content: flex-start \| flex-end \| center \| space-between \| space-around \| stretch;` | Aligns flex lines when there's extra space in the cross-axis |

## Flex Item Properties

| Property | Syntax | Description |
|----------|--------|-------------|
| order | `order: <integer>;` | Controls the order of flex items |
| flex-grow | `flex-grow: <number>;` | Determines how much an item will grow relative to other items |
| flex-shrink | `flex-shrink: <number>;` | Specifies how much an item will shrink relative to other items |
| flex-basis | `flex-basis: <length> \| auto;` | Sets the initial main size of a flex item |
| flex | `flex: <flex-grow> <flex-shrink> <flex-basis>;` | Shorthand for flex-grow, flex-shrink, and flex-basis |
| align-self | `align-self: auto \| flex-start \| flex-end \| center \| baseline \| stretch;` | Allows individual flex items to override the align-items value |

## Common Patterns

Center both horizontally and vertically:
```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

Equal-width columns:
```css
.container {
  display: flex;
}
.item {
  flex: 1;
}
```

Sticky footer:
```css
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
main {
  flex: 1;
}
```

Key Concepts:
- Main Axis: The primary axis along which flex items are laid out.
- Cross Axis: The axis perpendicular to the main axis.
- Flex Container: The parent element with display: flex;.
- Flex Items: The children of the flex container.

Flexbox Gotchas:
- Default flex-shrink: 1 can cause unexpected shrinking.
- flex-basis: auto uses the item's content size, which can be unpredictable.
- Margins don't collapse in a flex container.

Pro tip: Use the Firefox DevTools Flexbox Inspector for visual debugging of your flex layouts. It's a lifesaver when things get complex.