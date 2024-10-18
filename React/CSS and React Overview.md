## Naming Conventions

1. BEM (Block Element Modifier)
   - Block: `menu`
   - Element: `menu__item`
   - Modifier: `menu__item--active`

2. Component-Based Naming
   - Use component name as prefix: `ProfileButton-dropdown`

3. Functional/State Classes
   - Describe state or function: `is-hidden`, `is-loading`

4. Semantic Naming
   - Focus on purpose: `primary-button` (good), `blue-button` (avoid)

##### Practical Application:
Choose one primary naming convention for your project and stick to it consistently. BEM is excellent for larger projects with complex components, while component-based naming works well for simpler React applications. Combine your chosen primary convention with functional/state classes for dynamic styling. For example, you might use BEM for structural naming and add functional classes like 'is-active' for state changes. Always prioritize semantic, meaningful names over purely descriptive ones. Remember, the goal is to create a system that's intuitive for your team and scalable for your project.

See: [[Compare and Contrast - BEM + CSS Modules|BEM vs CSS Modules - Naming]]
## CSS Organization Methods

1. CSS Modules
   - Scoped styles for components
   - Usage: `import styles from './Component.module.css'`

2. Global Styles
   - App-wide styles and resets
   - Typically in `global.css` or `index.css`

3. Inline Styles
   - For dynamic, computed styles
   - Use sparingly: `style={{color: dynamicColor}}`

4. Styled Components (CSS-in-JS)
   - Combines CSS with component logic
   - Example: `const Button = styled.button`...``

## File Structure

```
src/
  components/
    Button/
      Button.js
      Button.module.css
  styles/
    global.css
    variables.css
    utilities.css
```

## Best Practices

1. Component Proximity
   - Keep styles close to components

2. Avoid Deep Nesting
   - Keeps specificity low, improves performance

3. Use Utility Classes
   - For common, reusable styles

4. Theme Variables
   - Use CSS variables for consistent theming

5. Responsive Design
   - Use media queries or responsive utilities

## Combining Approaches

- Primary: Choose CSS Modules or Styled Components for component-specific styles
- Secondary: Use global CSS for app-wide styles and utilities
- Tertiary: Add inline styles only for highly dynamic properties

## Example Integration

```jsx
// Button.module.css
.button {
  /* Base styles */
}
.button--primary {
  /* Primary styles */
}

// global.css
.text-center { text-align: center; }

// Button.js
import styles from './Button.module.css';
import './global.css';

function Button({ primary, style, children }) {
  return (
    <button 
      className={`${styles.button} ${primary ? styles['button--primary'] : ''} text-center`}
      style={style}
    >
      {children}
    </button>
  );
}
```

## Redux State in CSS

- Reflect Redux state in class names
- Example: `<div className={`user-profile ${isAuthenticated ? 'is-authenticated' : ''}`}>`

## Performance Considerations

- Minimize global styles
- Use CSS Modules to avoid class name collisions
- Be cautious with deeply nested selectors

## Accessibility

- Use semantic class names to enhance meaning
- Ensure sufficient color contrast
- Avoid using only color to convey information

## Responsive Design

- Use flexbox and grid for layouts
- Implement mobile-first approach
- Use relative units (em, rem) for scalability

## Tools and Libraries

- Sass or Less for advanced CSS features
- PostCSS for CSS processing
- Stylelint for CSS linting

Remember:
- Consistency is key - stick to one primary method
- Balance between reusability and specificity
- Consider team preferences and project requirements
- Start simple, add complexity as needed
