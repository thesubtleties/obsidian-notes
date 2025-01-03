
1. File Naming:
   - Use `.module.css` extension: `ComponentName.module.css`

2. Importing:
   ```javascript
   import styles from './ComponentName.module.css';
   ```

3. Usage in JSX:
   ```jsx
   <div className={styles.className}>
   ```

4. Classes vs IDs:
   - Classes: Preferred for most use cases
     ```css
     .className { /* styles */ }
     ```
     ```jsx
     <div className={styles.className}>
     ```
   - IDs: Use sparingly, mainly for JavaScript hooks or ARIA attributes
     ```css
     #idName { /* styles */ }
     ```
     ```jsx
     <div id={styles.idName}>
     ```

5. Why prefer classes over IDs:
   - Lower specificity, easier to override
   - Better aligned with component-based architecture
   - More flexible for reuse

6. Handling kebab-case:
   ```jsx
   <div className={styles['kebab-case-class']}>
   ```

7. [[Global Styles in CSS Modules]]:
   ```css
   :global(.globalClassName) { /* styles */ }
   ```

8. Composing styles:
   ```css
   .composed { composes: className1 className2; }
   ```

9. Vite-specific:
   - Built-in support, no configuration needed
   - Automatic TypeScript definitions generated

10. Benefits of CSS Modules:
    - Scoped styles, prevents conflicts
    - Explicit dependencies
    - Easier refactoring

11. Best Practices:
    - One CSS Module per component
    - Avoid element selectors (e.g., `div`, `span`)
    - Use camelCase for class names to avoid bracket notation in JSX

12. Debugging:
    - Generated class names include original name for easier debugging
    - Example: `ComponentName_className__uniqueHash`

13. Performance:
    - CSS Modules are processed at build time, no runtime cost

14. With React Portals:
    - CSS Modules work seamlessly, styles applied regardless of DOM position

15. Alternatives to IDs:
    - Use data attributes for JavaScript hooks:
      ```jsx
      <div className={styles.modalOverlay} data-testid="modal-overlay">
      ```

Remember:
- CSS Modules scope your styles to the component
- They work with most modern build tools (including Vite) out of the box
- They provide a good balance between global CSS and CSS-in-JS solutions

