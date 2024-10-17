## 1. Selectors

| Selector Type | Syntax | Description | Example |
|---------------|--------|-------------|---------|
| Element | `element { }` | Selects all elements of the specified type | `p { }` selects all `<p>` elements |
| Class | `.classname { }` | Selects elements with the specified class | `.highlight { }` selects elements with `class="highlight"` |
| ID | `#idname { }` | Selects the element with the specified id | `#header { }` selects the element with `id="header"` |
| Attribute | `[attribute="value"] { }` | Selects elements with the specified attribute and value | `[type="text"] { }` selects elements with `type="text"` |
| Descendant | `ancestor descendant { }` | Selects all descendant elements inside the ancestor | `div p { }` selects all `<p>` inside `<div>` |
| Child | `parent > child { }` | Selects all direct child elements of the parent | `ul > li { }` selects all `<li>` that are direct children of `<ul>` |
| Adjacent Sibling | `element1 + element2 { }` | Selects the first element2 that comes immediately after element1 | `h1 + p { }` selects the first `<p>` after an `<h1>` |
| Pseudo-class | `element:pseudo-class { }` | Selects and styles elements in a specific state | `a:hover { }` styles links when hovered over |
| Pseudo-element | `element::pseudo-element { }` | Selects and styles a part of an element | `p::first-line { }` styles the first line of all `<p>` |

## 2. Box Model

| Property | Syntax | Description | Example |
|----------|--------|-------------|---------|
| Width | `width: value;` | Sets the width of an element | `width: 100px;` |
| Height | `height: value;` | Sets the height of an element | `height: 100px;` |
| Padding | `padding: value;` | Sets the padding on all sides | `padding: 10px;` |
| Border | `border: width style color;` | Sets the border properties | `border: 1px solid black;` |
| Margin | `margin: value;` | Sets the margin on all sides | `margin: 10px;` |
| Box-sizing | `box-sizing: value;` | Defines how the total width and height of an element is calculated | `box-sizing: border-box;` |

## 3. Display

| Value | Syntax | Description |
|-------|--------|-------------|
| Block | `display: block;` | Makes the element a block-level element |
| Inline | `display: inline;` | Makes the element an inline element |
| Inline-block | `display: inline-block;` | Combines properties of both inline and block |
| None | `display: none;` | Hides the element completely |

## 4. Positioning

| Value | Syntax | Description |
|-------|--------|-------------|
| Static | `position: static;` | Default positioning, follows normal document flow |
| Relative | `position: relative;` | Positions relative to its normal position |
| Absolute | `position: absolute;` | Positions relative to nearest positioned ancestor |
| Fixed | `position: fixed;` | Positions relative to the viewport |
| Sticky | `position: sticky;` | Toggles between relative and fixed based on scroll position |

## 5. Colors

| Type | Syntax | Description | Example |
|------|--------|-------------|---------|
| Named | `color: name;` | Uses predefined color names | `color: red;` |
| Hex | `color: #RRGGBB;` | Uses hexadecimal color values | `color: #ff0000;` |
| RGB | `color: rgb(R, G, B);` | Uses Red, Green, Blue values (0-255) | `color: rgb(255, 0, 0);` |
| RGBA | `color: rgba(R, G, B, A);` | RGB with added Alpha channel for opacity | `color: rgba(255, 0, 0, 0.5);` |
| HSL | `color: hsl(H, S%, L%);` | Uses Hue, Saturation, Lightness values | `color: hsl(0, 100%, 50%);` |

## 6. Typography

| Property | Syntax | Description | Example |
|----------|--------|-------------|---------|
| Font Family | `font-family: value;` | Sets the font for text, with fallbacks | `font-family: Arial, sans-serif;` |
| Font Size | `font-size: value;` | Sets the size of the font | `font-size: 16px;` |
| Font Weight | `font-weight: value;` | Sets the thickness of characters in text | `font-weight: bold;` |
| Font Style | `font-style: value;` | Sets the style of the font | `font-style: italic;` |
| Text Align | `text-align: value;` | Aligns text within an element | `text-align: center;` |
| Text Decoration | `text-decoration: value;` | Adds decoration to text | `text-decoration: underline;` |
| Line Height | `line-height: value;` | Sets the height of a line of text | `line-height: 1.5;` |

## 7. Background

| Property | Syntax | Description | Example |
|----------|--------|-------------|---------|
| Background Color | `background-color: value;` | Sets the background color of an element | `background-color: #f0f0f0;` |
| Background Image | `background-image: url('');` | Sets an image as the background | `background-image: url('image.jpg');` |
| Background Repeat | `background-repeat: value;` | Controls how background image is repeated | `background-repeat: no-repeat;` |
| Background Position | `background-position: value;` | Sets the starting position of a background image | `background-position: center;` |
| Background Size | `background-size: value;` | Specifies the size of the background image | `background-size: cover;` |

## 8. Transitions

| Property | Syntax | Description |
|----------|--------|-------------|
| Transition | `transition: property duration timing-function delay;` | Defines a transition effect with all properties in one declaration |

## 9. Transforms

| Function | Syntax | Description |
|----------|--------|-------------|
| Translate | `transform: translateX(value);` | Moves an element horizontally |
| Rotate | `transform: rotate(value);` | Rotates an element |
| Scale | `transform: scale(value);` | Increases or decreases the size of an element |

## 10. Media Queries

| Syntax | Description |
|--------|-------------|
| `@media screen and (max-width: 600px) { }` | Applies styles based on device characteristics (like screen width) |

## 11. [[Flexbox]] (Basic)

| Property | Syntax | Description |
|----------|--------|-------------|
| Display | `display: flex;` | Creates a flex container |
| Flex Direction | `flex-direction: value;` | Sets the direction of flex items |
| Justify Content | `justify-content: value;` | Aligns flex items along the main axis |
| Align Items | `align-items: value;` | Aligns flex items along the cross axis |

## 12. [[Grid]] (Basic)

| Property | Syntax | Description |
|----------|--------|-------------|
| Display | `display: grid;` | Creates a grid container |
| Grid Template Columns | `grid-template-columns: value;` | Defines columns in a grid layout |
| Grid Gap | `grid-gap: value;` | Sets the gap between grid items |

## 13. Variables

| Syntax | Description |
|--------|-------------|
| `:root { --variable-name: value; }` | Defines a custom property (variable) |
| `element { property: var(--variable-name); }` | Uses a custom property (variable) |

## 14. Specificity

| Selector   | Specificity |
| ---------- | ----------- |
| !important | Highest     |
| Inline     | ↓           |
| ID         | ↓           |
| Class      | ↓           |
| Element    | Lowest      |

