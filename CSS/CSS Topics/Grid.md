## Grid Container Properties

| Property | Syntax | Description |
|----------|--------|-------------|
| display | `display: grid;` | Creates a grid container |
| grid-template-columns | `grid-template-columns: <track-size>...;` | Defines the columns of the grid |
| grid-template-rows | `grid-template-rows: <track-size>...;` | Defines the rows of the grid |
| grid-template-areas | `grid-template-areas: "<area-name>...";` | Defines named grid areas |
| grid-gap | `grid-gap: <row-gap> <column-gap>;` | Sets gaps (gutters) between rows and columns |
| justify-items | `justify-items: start \| end \| center \| stretch;` | Aligns grid items along the inline (row) axis |
| align-items | `align-items: start \| end \| center \| stretch;` | Aligns grid items along the block (column) axis |
| justify-content | `justify-content: start \| end \| center \| stretch \| space-around \| space-between \| space-evenly;` | Aligns the grid within the container along the inline axis |
| align-content | `align-content: start \| end \| center \| stretch \| space-around \| space-between \| space-evenly;` | Aligns the grid within the container along the block axis |

## Grid Item Properties

| Property | Syntax | Description |
|----------|--------|-------------|
| grid-column | `grid-column: <start-line> / <end-line>;` | Specifies a grid item's location based on column lines |
| grid-row | `grid-row: <start-line> / <end-line>;` | Specifies a grid item's location based on row lines |
| grid-area | `grid-area: <name> \| <row-start> / <column-start> / <row-end> / <column-end>;` | Specifies a grid item's size and location |
| justify-self | `justify-self: start \| end \| center \| stretch;` | Aligns a grid item inside a cell along the inline axis |
| align-self | `align-self: start \| end \| center \| stretch;` | Aligns a grid item inside a cell along the block axis |

## Special Units and Functions

- `fr` unit: Represents a fraction of the available space.
- `minmax()`: Sets a size range for track sizes.
- `repeat()`: Repeats a pattern of tracks.
- `auto-fit` and `auto-fill`: Creates a responsive grid without media queries.

## Key Concepts

- Grid Lines: The lines that make up the structure of the grid.
- Grid Tracks: The space between two adjacent grid lines.
- Grid Cell: The intersection of a row and a column.
- Grid Area: A rectangular space surrounded by four grid lines.

## Common Patterns

Basic responsive grid:
```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}
```

Grid with named areas:
```css
.container {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 200px 1fr;
}
```

Centering an item:
```css
.container {
  display: grid;
  place-items: center;
}
```

## Grid Gotchas

- Grid items don't collapse margins.
- Auto sized tracks will grow to fit content, which can lead to unexpected layouts.
- Implicit tracks are created automatically if you place an item outside the defined grid.

Pro tip: Use Firefox DevTools Grid Inspector for visual debugging of your grid layouts. It's incredibly helpful for understanding complex grids.

Remember, CSS Grid is powerful for both simple and complex layouts. It works great in combination with Flexbox for comprehensive layout control. Practice and experimentation are key to mastering Grid.