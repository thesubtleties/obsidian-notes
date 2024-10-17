## File Structure
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   └── Button.module.css
│   └── Header/
│       ├── Header.jsx
│       └── Header.module.css
├── styles/
│   ├── global.css
│   └── variables.css
├── App.jsx
└── main.jsx
```

## CSS Modules (Component-Specific Styling)

1. Naming:
   - Use `.module.css` extension (e.g., `Button.module.css`)

2. In CSS file:
```css
.button {
  background-color: var(--primary-color);
}
.buttonPrimary {
  composes: button;
  color: white;
}
```

3. In JSX:
```jsx
import styles from './Button.module.css';

function Button() {
  return <button className={styles.buttonPrimary}>Click me</button>;
}
```

4. Multiple Classes:
```jsx
className={`${styles.button} ${styles.large}`}
// or
className={[styles.button, styles.large].join(' ')}
```

## Global CSS

1. Create `src/styles/global.css`:
```css
:root {
  --primary-color: #007bff;
}

body {
  font-family: Arial, sans-serif;
}

.text-center {
  text-align: center;
}
```

2. Import in `main.jsx`:
```jsx
import './styles/global.css';
```

## CSS Variables (Custom Properties)

1. Define in global CSS:
```css
:root {
  --primary-color: #007bff;
  --font-size-large: 18px;
}
```

2. Use in any CSS file:
```css
.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-large);
}
```

## Sass with Vite

1. Install Sass:
```
npm install -D sass
```

2. Use `.scss` extension (e.g., `Button.module.scss`)

3. No additional configuration needed in Vite

## Conditional Styling

```jsx
<div className={isActive ? styles.active : styles.inactive}>
  Content
</div>
```

## CSS-in-JS (styled-components example)

1. Install:
```
npm install styled-components
```

2. Usage:
```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
`;

function Button({ primary }) {
  return <StyledButton primary={primary}>Click me</StyledButton>;
}
```

## Best Practices

1. Use CSS Modules for component-specific styles
2. Use global CSS for app-wide styles and utilities
3. Utilize CSS variables for theming and consistency
4. Keep styles close to their components
5. Use meaningful, descriptive class names
6. Consider BEM naming convention for global classes

## Vite-Specific Notes

1. CSS Modules work out-of-the-box
2. PostCSS is included by default
3. For custom CSS Modules config, use `vite.config.js`:
```javascript
export default {
  css: {
    modules: {
      generateScopedName: '[name]__[local]___[hash:base64:5]'
    }
  }
}
```

4. Hot Module Replacement (HMR) works automatically for CSS

Remember:
- Vite is designed for speed and simplicity
- Most CSS features work without additional configuration
- Leverage Vite's fast HMR for a smooth development experience

