
*Using Speaker/Team Member Cards as Example*

## 🎯 Core Concepts

### 1. Component Hierarchy
```
Base Component → Specialized Versions → Layout Wrapper → Page Implementation
(PersonCard)    (SpeakerCard)         (PersonGrid)      (SessionPage)
```

### 2. Breaking Down Components

**When to Break Down:**
- ✅ Component has multiple responsibilities
- ✅ UI element appears in multiple places
- ✅ Logic becomes complex
- ✅ Need for reusability
- ✅ Component exceeds 100-150 lines

**When to Keep Together:**
- 🚫 Components are tightly coupled
- 🚫 Would create prop drilling
- 🚫 Simple, self-contained piece
- 🚫 One-off UI element

### 3. Component Types

#### Base Components (Most Reusable)
```jsx
// Takes generic props, highly reusable
const PersonCard = ({ image, name, title }) => {...}
```
- Most generic version
- Accepts basic props
- No business logic
- Highly reusable
- Focused on presentation

#### Specialized Components
```jsx
// Uses base component with specific data structure
const SpeakerCard = ({ speaker }) => (
  <PersonCard
    image={speaker.headshot}
    name={speaker.name}
    title={speaker.title}
  />
);
```
- Adapts base component
- Handles specific data structures
- May add specific features
- Less reusable but more practical

#### Layout Components
```jsx
// Handles layout of multiple components
const PersonGrid = ({ children }) => (
  <div className="grid">{children}</div>
);
```
- Manages arrangement
- Uses children prop
- Purely for layout
- Highly reusable

### 4. Props Patterns

**Basic Props:**
```jsx
// Direct values
image="/path.jpg"
name="John"
```

**Object Props:**
```jsx
// Passing full objects
speaker={speakerData}
```

**Children Props:**
```jsx
// Passing components as children
<Grid>
  <Card />
  <Card />
</Grid>
```

### 5. Reusability Principles

**DRY (Don't Repeat Yourself):**
- Base components for shared UI
- Layout components for repeated patterns
- Consistent prop patterns

**Single Responsibility:**
- Each component does one thing
- Clear, focused purpose
- Easy to understand and maintain

**Composition over Configuration:**
```jsx
// Instead of:
<Card variant="speaker" size="large" />

// Better:
<SpeakerCard size="large" />
```

### 6. Common Patterns

**Container/Presenter Pattern:**
```jsx
// Container (Logic)
const SpeakerListContainer = () => {
  const [speakers] = useState([]);
  return <SpeakerList speakers={speakers} />;
};

// Presenter (UI)
const SpeakerList = ({ speakers }) => (
  <PersonGrid>
    {speakers.map(speaker => (
      <SpeakerCard speaker={speaker} />
    ))}
  </PersonGrid>
);
```

**Composition Pattern:**
```jsx
// Building complex UIs from simple pieces
<Page>
  <Section>
    <PersonGrid>
      <SpeakerCard />
    </PersonGrid>
  </Section>
</Page>
```

### 7. Best Practices

**Naming:**
- Clear and descriptive
- Indicates purpose
- Follows conventions
- PascalCase for components

**Props:**
- Clear prop names
- Default values when needed
- PropTypes or TypeScript
- Spread remaining props

**Structure:**
- Logical file organization
- Consistent patterns
- Clear component hierarchy
- Modular design

### 8. Red Flags 🚩

- Component does multiple things
- Too many props
- Deeply nested components
- Repeated code
- Unclear purpose
- Mixed responsibilities
- Hard to test

### 9. Testing Considerations

```jsx
// Components should be:
- Isolated
- Predictable
- Easy to test
- Clear inputs/outputs
```

### 10. Performance Tips

```jsx
// Use when needed:
- React.memo()
- useMemo()
- useCallback()
- Lazy loading
```

