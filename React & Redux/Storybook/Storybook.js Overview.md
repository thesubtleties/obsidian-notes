## 1. Installation & Setup
```bash
# Initialize Storybook in an existing project
npx storybook@latest init

# Start Storybook development server
npm run storybook

# Build Storybook for production
npm run build-storybook
```

## 2. Basic Story Structure
```javascript
/*
 * Basic Story Component Structure
 * Purpose: Demonstrates the fundamental way to write stories
 * Location: Usually in [ComponentName].stories.js/tsx
 */

// Import necessary dependencies
import { MyComponent } from './MyComponent';

// Define metadata for the component
export default {
  title: 'Components/MyComponent', // Navigation hierarchy
  component: MyComponent,
  // Optional parameters
  parameters: {
    layout: 'centered', // Story layout
  },
  // Optional decorators
  decorators: [
    (Story) => (
      <div style={{ padding: '2rem' }}>
        <Story />
      </div>
    ),
  ],
};

// Basic story
export const Default = {
  args: {
    label: 'Click me',
    onClick: () => console.log('clicked'),
  },
};

// Story with different props
export const Primary = {
  args: {
    ...Default.args,
    variant: 'primary',
  },
};
```

## 3. Args (Props) Management
```javascript
/*
 * Args Management Examples
 * Purpose: Shows different ways to handle component props
 */

// Template approach
const Template = (args) => <MyComponent {...args} />;

// Basic args
export const Default = Template.bind({});
Default.args = {
  label: 'Button',
  disabled: false,
};

// Args composition
export const Disabled = Template.bind({});
Disabled.args = {
  ...Default.args,
  disabled: true,
};

// Dynamic args
export const Dynamic = {
  args: {
    value: 'Dynamic Value',
    onChange: (e) => console.log('Value changed:', e.target.value),
  },
};
```

## 4. Decorators
```javascript
/*
 * Decorator Examples
 * Purpose: Wrapping stories with additional context or styling
 */

// Component-level decorator
export default {
  title: 'Components/MyComponent',
  component: MyComponent,
  decorators: [
    (Story) => (
      <div style={{ margin: '2em' }}>
        <Story />
      </div>
    ),
  ],
};

// Story-level decorator
export const WithBackground = {
  decorators: [
    (Story) => (
      <div style={{ background: 'lightgray', padding: '1em' }}>
        <Story />
      </div>
    ),
  ],
};
```

## 5. Parameters
```javascript
/*
 * Parameters Configuration
 * Purpose: Configure story behavior and appearance
 */

export default {
  title: 'Components/MyComponent',
  component: MyComponent,
  parameters: {
    // Control background colors
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#333333' },
      ],
    },
    // Control viewport sizes
    viewport: {
      defaultViewport: 'mobile1',
    },
    // Control actions
    actions: { argTypesRegex: '^on.*' },
    // Control docs
    docs: {
      description: {
        component: 'Component description here',
      },
    },
  },
};
```

## 6. Controls
```javascript
/*
 * Controls Configuration
 * Purpose: Define how args can be manipulated in Storybook UI
 */

export default {
  title: 'Components/MyComponent',
  component: MyComponent,
  argTypes: {
    // Text input
    label: {
      control: 'text',
      description: 'Button label',
    },
    // Select dropdown
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
    },
    // Boolean toggle
    disabled: {
      control: 'boolean',
    },
    // Color picker
    backgroundColor: {
      control: 'color',
    },
    // Number input
    padding: {
      control: { type: 'number', min: 0, max: 100, step: 1 },
    },
  },
};
```

## 7. Documentation
```javascript
/*
 * Documentation Examples
 * Purpose: Adding comprehensive documentation to stories
 */

import { Meta, Story, Canvas } from '@storybook/blocks';

// Component documentation
/**
 * This is a component description that will appear in docs
 */
export default {
  title: 'Components/MyComponent',
  component: MyComponent,
  parameters: {
    docs: {
      description: {
        component: 'A detailed description of the component',
      },
      source: {
        type: 'dynamic',
      },
    },
  },
};

// Story-level documentation
export const Example = {
  parameters: {
    docs: {
      description: {
        story: 'This is how you use the component in this specific case',
      },
    },
  },
};
```

## 8. Testing Integration
```javascript
/*
 * Testing Setup
 * Purpose: Integration with testing tools
 */

// Jest + Testing Library integration
import { render, screen } from '@testing-library/react';
import { composeStories } from '@storybook/testing-react';
import * as stories from './MyComponent.stories';

const { Default } = composeStories(stories);

test('renders component from story', () => {
  render(<Default />);
  expect(screen.getByText('Button')).toBeInTheDocument();
});
```

## 9. Addons Configuration
```javascript
// .storybook/main.js
module.exports = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
    // Custom addons
    'storybook-addon-designs',
  ],
  framework: {
    name: '@storybook/react-webpack5',
    options: {},
  },
};
```

## 10. Common Best Practices

1. **File Organization**
```plaintext
src/
  components/
    Button/
      Button.jsx
      Button.stories.jsx
      Button.test.jsx
      Button.css
```

2. **Naming Conventions**
```javascript
// Use clear, descriptive names for stories
export const DefaultButton = {...}
export const LargeDisabledButton = {...}
export const LoadingStateButton = {...}
```

1. **Component States**
```javascript
// Document all important states
export const Default = {...}
export const Hover = {...}
export const Active = {...}
export const Disabled = {...}
export const Loading = {...}
export const Error = {...}
```

## Practical Tips

2. **Development Workflow**
- Start Storybook alongside your main development server
- Develop components in isolation first
- Use hot reloading for rapid development

3. **Documentation**
- Write clear component descriptions
- Include usage examples
- Document props thoroughly
- Add notes about edge cases

4. **Testing**
- Use Storybook as a visual testing platform
- Integrate with Jest for unit testing
- Use interaction testing for complex components

5. **Performance**
- Lazy load stories when possible
- Use webpack optimization features
- Consider using Storybook's on-demand loading

Remember to:
- Keep stories simple and focused
- Document edge cases and variations
- Use consistent naming conventions
- Maintain organization as your storybook grows
- Regular updates and maintenance

This cheatsheet covers the fundamentals of Storybook.js. As you become more comfortable with these basics, you can explore more advanced features like:
- Custom addons development
- Advanced testing strategies
- CI/CD integration
- Theme customization
- Performance optimization