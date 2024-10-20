1. Basic Syntax:
   ```css
   :global(.className) { /* styles */ }
   ```

2. Purpose:
   - Declares styles that are not scoped to the current module
   - Applies styles globally across the entire application

3. Scope:
   - Affects all matching elements, regardless of which component's CSS Module it's defined in

4. Use Cases:
   - Styling third-party components
   - Creating app-wide utility classes
   - Defining reset or base styles

5. Behavior:
   - Bypasses CSS Modules' scoping mechanism
   - Acts like regular CSS, applying to all matching elements

6. Cascading:
   - Normal CSS cascading rules still apply
   - Can be overridden by more specific selectors

7. Syntax Variations:
   ```css
   :global {
     .className1 { /* styles */ }
     .className2 { /* styles */ }
   }
   ```

8. Mixing with Local Styles:
   ```css
   .localClass :global(.globalClass) { /* styles */ }
   ```

9. Best Practices:
   - Use sparingly to maintain modularity
   - Be specific with selectors to minimize unintended effects
   - Consider separate non-module CSS files for truly global styles

10. Potential Pitfalls:
    - Can lead to naming conflicts
    - May cause unexpected styling in other components
    - Can make styles harder to trace and debug

11. Alternatives:
    - Composition for shared styles between specific components
    - Separate global CSS file for app-wide styles

12. Performance:
    - No performance difference compared to regular CSS
    - Processed at build time, like other CSS Module styles

13. Debugging:
    - Global classes won't have unique identifiers in the DOM
    - Use browser dev tools to trace style applications

14. With Build Tools:
    - Supported out-of-the-box in most build setups (e.g., Vite, Create React App)
    - No additional configuration typically needed

Remember: While powerful, `:global` should be used judiciously. It's a tool for specific scenarios, not a default approach in CSS Modules.
