## 1. Basic Nested Component Structure
```javascript
/*
 * Nested Component Story Structure
 * Purpose: Demonstrates how to compose and document nested components
 * Location: src/stories/ComplexComponent.stories.jsx
 */

import { Card } from './Card';
import { CardHeader } from './CardHeader';
import { CardBody } from './CardBody';
import { CardFooter } from './CardFooter';

export default {
  title: 'Components/Card',
  component: Card,
  // Subcomponents registration
  subcomponents: { 
    CardHeader, 
    CardBody, 
    CardFooter 
  },
};

// Basic nested component story
export const CompleteCard = {
  render: () => (
    <Card>
      <CardHeader title="User Profile" />
      <CardBody>
        <UserInfo name="John Doe" />
        <UserStats activity={userActivityData} />
      </CardBody>
      <CardFooter>
        <Button>Edit Profile</Button>
      </CardFooter>
    </Card>
  ),
};
```

## 2. Page-Level Stories
```javascript
/*
 * Page Component Story
 * Purpose: Shows how to compose full pages from components
 * Location: src/stories/pages/UserDashboard.stories.jsx
 */

import { UserDashboard } from './UserDashboard';
import { Sidebar } from '../components/Sidebar';
import { MainContent } from '../components/MainContent';
import { Header } from '../components/Header';

export default {
  title: 'Pages/UserDashboard',
  component: UserDashboard,
  parameters: {
    // Disable default padding for full-page stories
    layout: 'fullscreen',
    // Optional: Define viewport sizes
    viewport: {
      defaultViewport: 'desktop',
    },
  },
};

// Complete page story
export const StandardDashboard = {
  render: () => (
    <UserDashboard>
      <Header 
        user={mockUser}
        notifications={mockNotifications}
      />
      <div className="dashboard-layout">
        <Sidebar 
          menuItems={mockMenuItems}
          activeItem="home"
        />
        <MainContent>
          <DashboardWidgets />
          <UserActivity />
          <RecentTransactions />
        </MainContent>
      </div>
    </UserDashboard>
  ),
};
```

## 3. Component Composition with Args
```javascript
/*
 * Component Composition with Props Control
 * Purpose: Shows how to manage props across nested components
 */

// Define the composed component story
export const ComposedCard = {
  args: {
    header: {
      title: 'Profile Settings',
      subtitle: 'Manage your account',
    },
    body: {
      userData: {
        name: 'Jane Smith',
        email: 'jane@example.com',
      },
    },
    footer: {
      primaryAction: 'Save Changes',
      secondaryAction: 'Cancel',
    },
  },
  render: (args) => (
    <Card>
      <CardHeader {...args.header} />
      <CardBody>
        <UserForm {...args.body} />
      </CardBody>
      <CardFooter {...args.footer} />
    </Card>
  ),
};
```

## 4. Context Providers for Nested Components
```javascript
/*
 * Context Provider Setup
 * Purpose: Demonstrates how to handle context in nested components
 */

import { ThemeProvider } from '../context/ThemeContext';
import { UserProvider } from '../context/UserContext';

// Global decorator for context
export default {
  title: 'Pages/UserDashboard',
  decorators: [
    (Story) => (
      <ThemeProvider theme="light">
        <UserProvider value={mockUserData}>
          <Story />
        </UserProvider>
      </ThemeProvider>
    ),
  ],
};

// Story with specific context override
export const DarkModeDashboard = {
  decorators: [
    (Story) => (
      <ThemeProvider theme="dark">
        <Story />
      </ThemeProvider>
    ),
  ],
  render: () => <DashboardComponent />,
};
```

## 5. Nested Component Documentation
```javascript
/*
 * Documentation for Nested Components
 * Purpose: Shows how to document component relationships
 */

import { Meta, Story, Canvas, ArgsTable } from '@storybook/blocks';

export default {
  title: 'Components/ComplexForm',
  component: ComplexForm,
  parameters: {
    docs: {
      description: {
        component: `
          # Complex Form Component
          This component is composed of multiple nested components:
          - FormHeader
          - FormFields
          - FormValidation
          - FormSubmit
          
          ## Component Hierarchy
          \`\`\`
          ComplexForm
          ├── FormHeader
          ├── FormFields
          │   ├── InputField
          │   ├── SelectField
          │   └── CheckboxGroup
          ├── FormValidation
          └── FormSubmit
          \`\`\`
        `,
      },
    },
  },
};
```

## 6. Testing Nested Components
```javascript
/*
 * Testing Nested Components
 * Purpose: Shows how to test composed components
 */

import { render, screen } from '@testing-library/react';
import { composeStories } from '@storybook/testing-react';
import * as stories from './ComplexForm.stories';

const { DefaultForm } = composeStories(stories);

test('renders complete form with all nested components', () => {
  render(<DefaultForm />);
  
  // Test header
  expect(screen.getByRole('heading')).toBeInTheDocument();
  
  // Test form fields
  expect(screen.getByLabelText('Username')).toBeInTheDocument();
  expect(screen.getByLabelText('Password')).toBeInTheDocument();
  
  // Test submit button
  expect(screen.getByRole('button', { name: 'Submit' })).toBeInTheDocument();
});
```

## 7. Best Practices for Nested Components

### Directory Structure
```plaintext
src/
  components/
    Card/
      index.jsx
      Card.jsx
      CardHeader.jsx
      CardBody.jsx
      CardFooter.jsx
      Card.stories.jsx
      __tests__/
        Card.test.jsx
    
  pages/
    Dashboard/
      index.jsx
      Dashboard.jsx
      Dashboard.stories.jsx
      components/
        DashboardHeader.jsx
        DashboardSidebar.jsx
        DashboardContent.jsx
```

### Component Organization Tips
1. **Composition Patterns**
```javascript
// Use composition over props drilling
const Dashboard = () => (
  <DashboardLayout>
    <DashboardHeader />
    <DashboardContent>
      {/* Nested components */}
      <Sidebar />
      <MainContent />
    </DashboardContent>
    <DashboardFooter />
  </DashboardLayout>
);
```

2. **Props Management**
```javascript
// Use prop spreading carefully
const CardWithContext = ({ children, ...props }) => (
  <Card {...props}>
    {React.Children.map(children, child => {
      // Pass context or modified props to children
      return React.cloneElement(child, { theme: props.theme });
    })}
  </Card>
);
```

1. **Story Organization**
```javascript
// Group related stories
export default {
  title: 'Components/Card',
  component: Card,
  subcomponents: { CardHeader, CardBody, CardFooter },
};

// Show individual components
export const Header = {
  render: () => <CardHeader title="Sample Header" />,
};

// Show composed components
export const FullCard = {
  render: () => (
    <Card>
      <CardHeader title="Title" />
      <CardBody>Content</CardBody>
      <CardFooter>Footer</CardFooter>
    </Card>
  ),
};
```

Remember:
- Keep components modular and reusable
- Document component relationships clearly
- Use consistent patterns for nested component communication
- Consider performance implications of deep nesting
- Implement proper error boundaries for nested components
- Use composition patterns that make sense for your use case
- Test components both in isolation and as part of the larger structure