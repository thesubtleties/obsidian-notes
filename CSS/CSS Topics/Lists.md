
```css
/* Basic List Styling */

/* Remove default padding and bullet points */
ul, ol {
  padding-left: 0;
  list-style-type: none;
}

/* Custom bullet points for unordered lists */
ul li::before {
  content: "•"; /* You can use any character or emoji */
  color: #fd9519; /* Your theme color */
  font-weight: bold;
  display: inline-block;
  width: 1em;
  margin-left: -1em;
}

/* Custom numbering for ordered lists */
ol {
  counter-reset: item;
}

ol li::before {
  content: counter(item) ". ";
  counter-increment: item;
  color: #fd9519; /* Your theme color */
  font-weight: bold;
}

/* Horizontal list */
.horizontal-list {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
}

.horizontal-list li {
  margin-right: 20px;
}

/* Nested lists */
ul ul, ol ol, ul ol, ol ul {
  margin-left: 20px;
}

/* Styled list with borders */
.bordered-list li {
  border-bottom: 1px solid #ccc;
  padding: 10px 0;
}

.bordered-list li:last-child {
  border-bottom: none;
}

/* List with hover effects */
.interactive-list li {
  transition: background-color 0.3s ease;
}

.interactive-list li:hover {
  background-color: rgba(253, 149, 25, 0.1); /* Light version of your theme color */
}

/* Icon list (assuming you're using a icon library like Font Awesome) */
.icon-list li::before {
  font-family: 'Font Awesome 5 Free';
  content: "\f005"; /* Star icon, change as needed */
  font-weight: 900;
  margin-right: 10px;
  color: #fd9519; /* Your theme color */
}

/* Two-column list */
.two-column-list {
  column-count: 2;
  column-gap: 40px;
}

/* Responsive two-column list */
@media (max-width: 768px) {
  .two-column-list {
    column-count: 1;
  }
}

/* Animated list items */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.animated-list li {
  animation: fadeIn 0.5s ease-out forwards;
  opacity: 0;
}

.animated-list li:nth-child(1) { animation-delay: 0.1s; }
.animated-list li:nth-child(2) { animation-delay: 0.2s; }
.animated-list li:nth-child(3) { animation-delay: 0.3s; }
/* Add more if needed */

/* Halloween-themed list (for your spooky theme) */
.spooky-list li::before {
  content: "🎃"; /* Pumpkin emoji, you can change to other Halloween-themed emojis */
  margin-right: 10px;
}
```

This CSS add-on provides various styling options for lists:

1. Basic styling for ul and ol with custom bullet points and numbering
2. Horizontal list layout
3. Styling for nested lists
4. Bordered list items
5. Interactive list with hover effects
6. Icon list (using Font Awesome as an example)
7. Two-column list layout with responsive design
8. Animated list items with a fade-in effect
9. A Halloween-themed list style using emojis

To use these styles, you can add the appropriate class to your list element. For example:

```html
<ul class="spooky-list">
  <li>Haunted House</li>
  <li>Ghostly Getaway</li>
  <li>Witch's Cottage</li>
</ul>
```

Remember to adjust colors, sizes, and other properties to match your overall design theme. Also, when using icon fonts like Font Awesome, make sure you've included the necessary library in your project.