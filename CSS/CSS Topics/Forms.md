# CSS Forms Styling Cheat Sheet

## Basic Form Structure
```css
/* Form container */
form {
    width: 100%;
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    background: #f9f9f9;
    border-radius: 8px;
}

.form-group {
    margin-bottom: 1rem;
    display: flex;
    flex-direction: column;
}

label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
    color: #333;
}
```

> [!example] Live Example
> <form style="width: 100%; max-width: 500px; margin: 0 auto; padding: 20px; background: #f9f9f9; border-radius: 8px;">
>   <div style="margin-bottom: 1rem; display: flex; flex-direction: column;">
>     <label style="display: block; margin-bottom: 0.5rem; font-weight: 500; color: #333;">Example Input</label>
>     <input type="text" placeholder="Type something...">
>   </div>
> </form>

## Input Fields
```css
input[type="text"],
input[type="email"],
input[type="password"] {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
    line-height: 1.5;
}

input:focus {
    outline: none;
    border-color: #4A90E2;
    box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.2);
}
```

> [!example] Live Example
> <div style="display: flex; flex-direction: column; gap: 10px; background: #f9f9f9; padding: 20px; border-radius: 8px;">
>   <input type="text" placeholder="Text input" style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; line-height: 1.5;">
>   <input type="email" placeholder="Email input" style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; line-height: 1.5;">
>   <input type="password" placeholder="Password input" style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; line-height: 1.5;">
> </div>

## Textarea
```css
textarea {
    width: 100%;
    min-height: 100px;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    resize: vertical;
}
```

> [!example] Live Example
> <textarea style="width: 100%; min-height: 100px; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; resize: vertical;" placeholder="Type your message here..."></textarea>

## Select Dropdown
```css
select {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: white;
    cursor: pointer;
}
```

> [!example] Live Example
> <select style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; background color: white; cursor: pointer;">
>   <option>Select an option</option>
>   <option>Option 1</option>
>   <option>Option 2</option>
>   <option>Option 3</option>
> </select>

## Buttons
```css
button,
input[type="submit"] {
    padding: 10px 20px;
    background-color: #4A90E2;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s;
}

button:hover {
    background-color: #357ABD;
}

button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}
```

> [!example] Live Example
> <div style="display: flex; gap: 10px;">
>   <button style="padding: 10px 20px; background-color: #4A90E2; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 16px;">Normal Button</button>
>   <button style="padding: 10px 20px; background-color: #ccc; color: white; border: none; border-radius: 4px; cursor: not-allowed; font-size: 16px;" disabled>Disabled Button</button>
> </div>

## Checkbox and Radio
```css
.checkbox-wrapper,
.radio-wrapper {
    display: flex;
    align-items: center;
    gap: 8px;
}

input[type="checkbox"],
input[type="radio"] {
    width: 16px;
    height: 16px;
    margin-right: 8px;
}
```

> [!example] Live Example
> <div style="display: flex; flex-direction: column; gap: 10px; background: #f9f9f9; padding: 20px; border-radius: 8px;">
>   <div style="display: flex; align-items: center; gap: 8px;">
>     <input type="checkbox" style="width: 16px; height: 16px; margin-right: 8px;">
>     <label>Checkbox example</label>
>   </div>
>   <div style="display: flex; align-items: center; gap: 8px;">
>     <input type="radio" name="radio-example" style="width: 16px; height: 16px; margin-right: 8px;">
>     <label>Radio 1</label>
>   </div>
>   <div style="display: flex; align-items: center; gap: 8px;">
>     <input type="radio" name="radio-example" style="width: 16px; height: 16px; margin-right: 8px;">
>     <label>Radio 2</label>
>   </div>
> </div>

## Complete Form Example
```html
<form class="example-form">
    <div class="form-group">
        <label>Name</label>
        <input type="text" placeholder="Enter your name">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input type="email" placeholder="Enter your email">
    </div>
    <div class="form-group">
        <label>Message</label>
        <textarea placeholder="Your message"></textarea>
    </div>
    <button type="submit">Submit</button>
</form>
```

> [!example] Live Example
> <form style="width: 100%; max-width: 500px; margin: 0 auto; padding: 20px; background: #f9f9f9; border-radius: 8px;">
>     <div style="margin-bottom: 1rem; display: flex; flex-direction: column;">
>         <label style="margin-bottom: 0.5rem; font-weight: 500;">Name</label>
>         <input type="text" placeholder="Enter your name" style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px;">
>     </div>
>     <div style="margin-bottom: 1rem; display: flex; flex-direction: column;">
>         <label style="margin-bottom: 0.5rem; font-weight: 500;">Email</label>
>         <input type="email" placeholder="Enter your email" style="width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px;">
>     </div>
>     <div style="margin-bottom: 1rem; display: flex; flex-direction: column;">
>         <label style="margin-bottom: 0.5rem; font-weight: 500;">Message</label>
>         <textarea placeholder="Your message" style="width: 100%; min-height: 100px; padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px;"></textarea>
>     </div>
>     <button type="submit" style="padding: 10px 20px; background-color: #4A90E2; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 16px;">Submit</button>
> </form>

## Key Points & Best Practices
- Always provide visual feedback for user interactions
- Maintain consistent padding and spacing
- Use appropriate input types
- Include focus states for accessibility
- Ensure sufficient contrast for readability
- Use `font-size: 16px` on mobile to prevent zoom on iOS
- Implement proper error handling and validation feedback
- Consider mobile-first design
- Use semantic HTML elements

Note: Some interactive features (like `:hover` states) may not be fully demonstrable in Obsidian's preview, but they will work in a web browser.