## Attribute Naming

HTML/CSS | React
---------|------
class | className
for (in labels) | htmlFor
onclick | onClick
onsubmit | onSubmit
style | style (as an object)

## Event Handlers

HTML | React
-----|------
onclick="handleClick()" | onClick={handleClick}
onsubmit="handleSubmit()" | onSubmit={handleSubmit}
onchange="handleChange()" | onChange={handleChange}
onkeydown="handleKeyDown()" | onKeyDown={handleKeyDown}
onmouseover="handleMouseOver()" | onMouseOver={handleMouseOver}

## Inline Styles

| CSS                       | React                    |
| ------------------------- | ------------------------ |
| font-size: 16px;          | fontSize: '16px',        |
| background-color: `#fff`; | backgroundColor: '#fff', |
| margin-top: 10px;         | marginTop: '10px',       |
| text-align: center;       | textAlign: 'center',     |

## Example of Inline Styles in React

```jsx
<div style={{
  fontSize: '16px',
  backgroundColor: '#fff',
  marginTop: '10px',
  textAlign: 'center'
}}>
  Content
</div>
```

## Form Elements

HTML | React
-----|------
`<input value="text" />` | `<input value={text} onChange={handleChange} />`
`<textarea>Text</textarea>` | `<textarea value={text} onChange={handleChange} />`
`<select>` | `<select value={value} onChange={handleChange}>`

## Conditional Rendering

```jsx
{condition && <ComponentA />}
{condition ? <ComponentA /> : <ComponentB />}
```

## List Rendering

```jsx
<ul>
  {items.map(item => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
```

## Data Attributes

HTML | React
-----|------
data-id="123" | data-id="123" (same in React)

## SVG Properties

HTML | React
-----|------
class | className
stroke-width | strokeWidth
fill-opacity | fillOpacity

## Special Cases

HTML | React
-----|------
tabindex | tabIndex
readonly | readOnly
maxlength | maxLength
cellpadding | cellPadding
cellspacing | cellSpacing
colspan | colSpan
rowspan | rowSpan

Remember:
- In React, use camelCase for property names.
- Event handlers in React take functions, not strings.
- Style properties in React are objects with camelCased keys.
- Boolean attributes (like disabled) are set to {true} or {false} in React.

This cheatsheet covers many common conversions, but React has many more nuances. Always refer to the official React documentation for the most up-to-date and comprehensive information.