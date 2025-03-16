# The Ultimate Guide to Building a Modern Next.js Application with Tailwind CSS v4

## Introduction

This guide walks through the process of building a professional, maintainable Next.js application from scratch, using Tailwind CSS v4, TypeScript, and a component-based architecture. We'll cover everything from initial setup to best practices for organization and development.

## Table of Contents

1. [Project Setup](#1-project-setup)
2. [Tailwind CSS v4 Configuration](#2-tailwind-css-v4-configuration)
3. [Project Structure and Organization](#3-project-structure-and-organization)
4. [Understanding Next.js App Router](#4-understanding-nextjs-app-router)
5. [Component-Based Architecture](#5-component-based-architecture)
6. [TypeScript Integration](#6-typescript-integration)
7. [Building Pages and Features](#7-building-pages-and-features)
8. [Styling with Tailwind CSS v4](#8-styling-with-tailwind-css-v4)
9. [Form Handling](#9-form-handling)
10. [Best Practices and Lessons Learned](#10-best-practices-and-lessons-learned)

## 1. Project Setup

### Creating a New Next.js Project

Start by creating a new Next.js project with TypeScript support:

```bash
npx create-next-app@latest my-project --typescript --eslint --app
cd my-project
```

This command:
- Creates a new Next.js project
- Enables TypeScript
- Sets up ESLint
- Uses the App Router (instead of the older Pages Router)
-  npx create-next-app@latest my-project will start the process and ask you questions on what features you want

### Understanding the Initial Project Structure

After creation, your project will have this basic structure:

```
/my-project
  /app
    layout.tsx
    page.tsx
  /public
  package.json
  next.config.js
  tsconfig.json
```

## 2. Tailwind CSS v4 Configuration

### Installing Tailwind CSS v4

Install Tailwind CSS v4 and its dependencies:

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

### Setting Up PostCSS Configuration

Create a `postcss.config.mjs` file in the root of your project:

```javascript
// postcss.config.mjs
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```

This configuration tells PostCSS to use the Tailwind CSS v4 plugin.

### Installing Tailwind Plugins

Install additional Tailwind plugins for enhanced functionality:

```bash
npm install @tailwindcss/typography @tailwindcss/forms
```

### Creating a Minimal Tailwind Configuration

Create a `tailwind.config.js` file for plugin configuration:

```javascript
// tailwind.config.js
module.exports = {
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms')
  ],
};
```

### Setting Up Your CSS File

Create or modify your `globals.css` file:

```css
/* globals.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";

@theme {
  /* Custom Colors */
  --color-greek-blue: oklch(0.65 0.15 240); /* #4A90E2 */
  --color-greek-gold: oklch(0.76 0.18 85); /* #D4AF37 */
  --color-greek-white: oklch(0.97 0 0); /* #F5F5F5 */
  --color-greek-gray: oklch(0.27 0 0); /* #333333 */
  
  /* Custom Fonts */
  --font-sans: var(--font-sans);
  --font-serif: var(--font-serif);
  --font-accent: var(--font-display);
  
  /* Custom Breakpoints */
  --breakpoint-xs: 30rem; /* 480px */
  --breakpoint-sm: 40rem; /* 640px */
  --breakpoint-md: 48rem; /* 768px */
  --breakpoint-lg: 64rem; /* 1024px */
  --breakpoint-xl: 80rem; /* 1280px */
  --breakpoint-2xl: 96rem; /* 1536px */
  
  /* Custom Spacing */
  --spacing-72: 18rem;
  --spacing-84: 21rem;
  --spacing-96: 24rem;
  
  /* Custom Border Radius */
  --radius-xl: 1rem;
  --radius-2xl: 2rem;
}

/* Custom utilities */
@utility select-greek {
  @apply rounded-md border-greek-white/20 bg-black/30 text-greek-white;
  @apply focus:border-greek-gold focus:ring focus:ring-greek-gold/30;
  
  option {
    background-color: #1f2937;
    color: white;
  }
}

@utility date-input-greek {
  &::-webkit-calendar-picker-indicator {
    filter: invert(1);
  }
}

/* Base styles */
body {
  background: var(--background);
  color: var(--foreground);
  font-family: var(--font-sans);
}
```

### Import CSS in Your Layout

Make sure to import your CSS file in your root layout:

```tsx
// app/layout.tsx
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

## 3. Project Structure and Organization

### Planning Your Project Structure

Before diving into coding, plan your project structure. Here's the structure we developed:

```
/src
  /app                  # Routes and page components
    /(main)             # Route group for main pages
      /about
        page.tsx
      /contact
        page.tsx
      /menu
        page.tsx
      /reservations
        page.tsx
      layout.tsx        # Shared layout for main pages
      page.tsx          # Home page
  /components           # Reusable components
    /ui                 # Generic UI components
      Card.tsx
      PageHeader.tsx
      FormInput.tsx
    /menu               # Menu-specific components
      MenuCard.tsx
      MenuCategories.tsx
    /contact            # Contact-specific components
      ContactForm.tsx
      ContactInfo.tsx
  /lib                  # Utility functions and services
    menuService.ts
    api.ts
  /types                # TypeScript type definitions
    menu.ts
    reservation.ts
  /public               # Static assets
    /images
```

### Why This Structure?

1. **Separation of Concerns**: Each directory has a specific purpose
2. **Modularity**: Components are grouped by feature or function
3. **Scalability**: Easy to add new features without cluttering existing code
4. **Discoverability**: New team members can quickly understand where things are

### Creating the Directory Structure

Create these directories to establish your project structure:

```bash
mkdir -p src/app/(main)/{about,contact,menu,reservations}
mkdir -p src/components/{ui,menu,contact}
mkdir -p src/lib
mkdir -p src/types
```

## 4. Understanding Next.js App Router

### How App Router Works

Next.js App Router uses a file-system based router where:

1. **Folders define routes**: Each folder in the `/app` directory becomes a route segment
2. **page.tsx files make routes accessible**: A route is only publicly accessible if it has a page.tsx file
3. **layout.tsx files define shared UI**: Layouts wrap pages and persist across navigations
4. **Route groups with (parentheses)**: Folders with names in parentheses don't affect the URL path

### Our Route Structure

In our project:

- `/app/(main)/page.tsx` → Home page (`/`)
- `/app/(main)/about/page.tsx` → About page (`/about`)
- `/app/(main)/contact/page.tsx` → Contact page (`/contact`)
- `/app/(main)/menu/page.tsx` → Menu page (`/menu`)
- `/app/(main)/reservations/page.tsx` → Reservations page (`/reservations`)

The `(main)` folder is a route group that doesn't affect the URL path but allows these pages to share a common layout.

### Creating a Shared Layout

```tsx
// app/(main)/layout.tsx
import Header from '@/components/Header';
import Footer from '@/components/Footer';

export default function MainLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex flex-col min-h-screen">
      <Header />
      <main className="flex-grow">
        {children}
      </main>
      <Footer />
    </div>
  );
}
```

## 5. Component-Based Architecture

### The Philosophy of Component-Based Architecture

Component-based architecture breaks down the UI into reusable, self-contained pieces. This approach:

1. **Improves Reusability**: Components can be used across different pages
2. **Enhances Maintainability**: Changes to a component affect all instances
3. **Facilitates Collaboration**: Team members can work on different components simultaneously
4. **Simplifies Testing**: Components can be tested in isolation

### Types of Components in Our Project

We created several types of components:

1. **UI Components**: Generic, highly reusable components like Card, PageHeader, FormInput
2. **Feature Components**: Components specific to a feature, like MenuCard or ContactForm
3. **Layout Components**: Components that define the structure of pages, like Header and Footer
4. **Page Components**: Components that represent entire pages

### Example: Creating a Reusable Card Component

```tsx
// components/ui/Card.tsx
import { ReactNode } from 'react';

interface CardProps {
  children: ReactNode;
  className?: string;
}

export default function Card({ children, className = '' }: CardProps) {
  return (
    <div className={`rounded-lg bg-black/30 backdrop-blur-sm p-8 shadow-sm ${className}`}>
      {children}
    </div>
  );
}
```

### Example: Creating a Feature-Specific Component

```tsx
// components/menu/MenuCard.tsx
import Image from 'next/image';
import Card from '@/components/ui/Card';
import { MenuItem } from '@/types/menu';

interface MenuCardProps {
  item: MenuItem;
}

export default function MenuCard({ item }: MenuCardProps) {
  return (
    <Card className="overflow-hidden">
      <div className="relative h-56 w-full">
        <Image
          src={item.image.url}
          alt={item.image.alt}
          fill
          className="object-cover"
        />
        {item.isVegetarian && (
          <span className="absolute top-3 left-3 bg-green-800/80 text-white px-3 py-1 text-xs">
            Vegetarian
          </span>
        )}
      </div>
      <div className="p-6">
        <h3 className="font-serif text-xl text-greek-white">{item.name}</h3>
        <p className="text-greek-white/80">{item.description}</p>
        <span className="text-greek-gold">${item.price.toFixed(2)}</span>
      </div>
    </Card>
  );
}
```

## 6. TypeScript Integration

### Benefits of TypeScript

TypeScript adds static typing to JavaScript, which:

1. **Catches Errors Early**: Type errors are caught at compile time, not runtime
2. **Improves Developer Experience**: Better autocomplete and documentation
3. **Makes Refactoring Safer**: The compiler catches breaking changes
4. **Enhances Code Quality**: Forces you to think about data structures

### Creating Type Definitions

For complex data structures, create dedicated type files:

```tsx
// types/menu.ts
export interface Category {
  id: string;
  name: string;
  order: number;
}

export interface MenuItem {
  id: string;
  name: string;
  description: string;
  price: number;
  image: {
    url: string;
    alt: string;
  };
  category: Category;
  isVegetarian: boolean;
  isSpicy: boolean;
  isAvailable: boolean;
  ingredients: Array<{ ingredient: string }>;
}
```

### Using Types in Components

```tsx
// components/menu/MenuCategories.tsx
import { Category } from '@/types/menu';

interface MenuCategoriesProps {
  categories: Category[];
  selectedCategory?: string;
  onCategoryChange: (categoryId?: string) => void;
}

export default function MenuCategories({ 
  categories, 
  selectedCategory, 
  onCategoryChange 
}: MenuCategoriesProps) {
  // Component implementation
}
```

### Typing Component Props

For simpler components, define types inline:

```tsx
// components/ui/PageHeader.tsx
interface PageHeaderProps {
  title: string;
  subtitle?: string;
}

export default function PageHeader({ title, subtitle }: PageHeaderProps) {
  // Component implementation
}
```

## 7. Building Pages and Features

### Creating a Basic Page

```tsx
// app/(main)/about/page.tsx
import { Metadata } from 'next';
import PageHeader from '@/components/ui/PageHeader';
import Card from '@/components/ui/Card';

export const metadata: Metadata = {
  title: 'About - Odyssey By Sely',
  description: 'Learn about our restaurant and our story.',
};

export default function AboutPage() {
  return (
    <div className="container mx-auto px-4 py-16">
      <PageHeader
        title="Our Story"
        subtitle="Discover the passion and philosophy behind Odyssey By Sely"
      />
      
      <div className="mt-16">
        <Card>
          <p className="text-greek-white/80">
            Founded in 2018, Odyssey By Sely began as a dream to bring authentic 
            Mediterranean flavors with a modern French twist to San Francisco.
          </p>
          {/* More content */}
        </Card>
      </div>
    </div>
  );
}
```

### Building a Feature: Contact Form

Break down complex features into smaller components:

```tsx
// app/(main)/contact/page.tsx
import { Metadata } from 'next';
import PageHeader from '@/components/ui/PageHeader';
import ContactForm from '@/components/contact/ContactForm';
import ContactInfo from '@/components/contact/ContactInfo';
import Card from '@/components/ui/Card';

export const metadata: Metadata = {
  title: 'Contact - Odyssey By Sely',
  description: 'Get in touch with us for reservations, inquiries, or feedback.',
};

export default function ContactPage() {
  return (
    <div className="container mx-auto px-4 py-16">
      <PageHeader
        title="Contact Us"
        subtitle="We'd love to hear from you. Reach out with any questions or feedback."
      />

      <div className="mt-16 grid gap-8 md:grid-cols-2">
        <Card>
          <h2 className="text-2xl font-serif font-bold text-greek-white">Send Us a Message</h2>
          <ContactForm />
        </Card>

        <div className="space-y-8">
          <Card>
            <h2 className="text-2xl font-serif font-bold text-greek-white">Contact Information</h2>
            <ContactInfo />
          </Card>
          {/* More cards */}
        </div>
      </div>
    </div>
  );
}
```

## 8. Styling with Tailwind CSS v4

### Using Theme Variables

Tailwind CSS v4 allows you to define theme variables in your CSS:

```css
@theme {
  --color-greek-blue: oklch(0.65 0.15 240);
  --color-greek-gold: oklch(0.76 0.18 85);
}
```

Then use them in your components:

```tsx
<button className="bg-greek-blue text-greek-white hover:bg-greek-gold">
  Click Me
</button>
```

### Creating Custom Utilities

For complex styling patterns, create custom utilities:

```css
@utility select-greek {
  @apply rounded-md border-greek-white/20 bg-black/30 text-greek-white;
  @apply focus:border-greek-gold focus:ring focus:ring-greek-gold/30;
  
  option {
    background-color: #1f2937;
    color: white;
  }
}
```

Then use them in your components:

```tsx
<select className="select-greek mt-1 block w-full">
  {options.map(option => (
    <option key={option.value} value={option.value}>{option.label}</option>
  ))}
</select>
```

### Using the New Opacity Syntax

Tailwind CSS v4 uses a new slash syntax for opacity:

```tsx
{/* Old way (v3) */}
<div className="bg-black bg-opacity-40"></div>

{/* New way (v4) */}
<div className="bg-black/40"></div>
```

### Leveraging Tailwind Plugins

The Typography plugin adds a set of prose classes for styling content:

```tsx
<article className="prose prose-invert">
  <h1>Article Title</h1>
  <p>This is a paragraph with some <strong>bold text</strong> and <em>italic text</em>.</p>
</article>
```

The Forms plugin improves the default styling of form elements:

```tsx
<input type="text" className="form-input-greek mt-1 block w-full" />
<textarea className="form-textarea-greek mt-1 block w-full" rows={4}></textarea>
```

## 9. Form Handling

### Creating Reusable Form Components

Build reusable form components to maintain consistency:

```tsx
// components/ui/FormInput.tsx
import { InputHTMLAttributes } from 'react';

interface FormInputProps extends InputHTMLAttributes<HTMLInputElement | HTMLTextAreaElement> {
  label: string;
  id: string;
  isTextarea?: boolean;
  rows?: number;
}

export default function FormInput({ 
  label, 
  id, 
  isTextarea = false, 
  rows = 6, 
  ...props 
}: FormInputProps) {
  const baseClassName = "mt-1 block w-full rounded-md border-greek-white/20 bg-black/30 text-greek-white focus:border-greek-gold focus:ring-greek-gold focus:ring-opacity-50";
  
  return (
    <div>
      <label htmlFor={id} className="block text-sm font-medium text-greek-white">
        {label}
      </label>
      {isTextarea ? (
        <textarea
          id={id}
          rows={rows}
          className={baseClassName}
          {...props as InputHTMLAttributes<HTMLTextAreaElement>}
        />
      ) : (
        <input
          id={id}
          className={props.type === 'date' ? "date-input-greek mt-1 block w-full" : baseClassName}
          {...props as InputHTMLAttributes<HTMLInputElement>}
        />
      )}
    </div>
  );
}
```

### Building a Form Component

Use the reusable form components to build a complete form:

```tsx
// components/reservations/ReservationForm.tsx
'use client';

import { useState } from 'react';
import FormInput from '@/components/ui/FormInput';
import FormSelect from '@/components/ui/FormSelect';

export default function ReservationForm() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [success, setSuccess] = useState(false);
  
  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    // Form submission logic
    
    setSuccess(true);
    setIsSubmitting(false);
  };
  
  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      <FormInput
        label="Full Name"
        id="name"
        name="name"
        type="text"
        required
      />
      
      <div className="grid grid-cols-1 gap-6 md:grid-cols-2">
        <FormInput
          label="Email Address"
          id="email"
          name="email"
          type="email"
          required
        />
        
        <FormInput
          label="Phone Number"
          id="phone"
          name="phone"
          type="tel"
          required
        />
      </div>
      
      {/* More form fields */}
      
      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full rounded-md bg-greek-blue px-8 py-3 text-greek-white hover:bg-greek-gold transition-colors"
      >
        {isSubmitting ? 'Submitting...' : 'Confirm Reservation'}
      </button>
    </form>
  );
}
```

### Handling Form State

Use React's state management to handle form state:

```tsx
'use client';

import { useState } from 'react';

export default function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  // Form submission logic
  
  return (
    <form>
      {/* Form fields that use handleChange */}
    </form>
  );
}
```

## 10. Best Practices and Lessons Learned

### Planning Before Coding

Taking time to plan your project structure before diving into code pays off in the long run:

1. **Identify Key Features**: List all the features your application needs
2. **Define Data Models**: Outline the data structures you'll work with
3. **Plan Component Hierarchy**: Sketch how components will be organized
4. **Establish Naming Conventions**: Decide on consistent naming patterns

### Benefits of Strong Componentization

Strong componentization offers numerous benefits:

1. **Modularity**: Components can be developed and tested in isolation
2. **Reusability**: Well-designed components can be reused across the application
3. **Maintainability**: Changes to a component affect all instances, reducing duplication
4. **Collaboration**: Team members can work on different components simultaneously
5. **Testing**: Isolated components are easier to test

### Organizing Components by Feature

Organizing components by feature rather than type:

```
/components
  /ui             # Generic UI components
  /menu           # Menu-specific components
  /contact        # Contact-specific components
  /reservations   # Reservation-specific components
```

This approach:
- Makes it easier to find related components
- Reduces the need to navigate between directories
- Allows for better code organization as the project grows

### Consistent Naming Conventions

Establish and follow consistent naming conventions:

1. **PascalCase for Components**: `MenuCard.tsx`, `FormInput.tsx`
2. **camelCase for Functions and Variables**: `handleSubmit`, `isLoading`
3. **kebab-case for CSS Classes**: `bg-greek-blue`, `text-greek-white`
4. **Descriptive Names**: Prefer descriptive names over abbreviations

### Avoiding Prop Drilling

As your component hierarchy grows, avoid prop drilling by:

1. **Restructuring Components**: Keep related state close to where it's used
2. **Using Context API**: For state that needs to be accessed by many components
3. **Using State Management Libraries**: For complex state management needs

### Progressive Enhancement

Build your application with progressive enhancement in mind:

1. **Start with Core Functionality**: Ensure the basic functionality works first
2. **Add Enhancements Incrementally**: Add animations, transitions, and advanced features later
3. **Test Across Devices**: Ensure your application works on various devices and browsers

### Documentation

Document your code and components:

1. **README Files**: Provide overview and setup instructions
2. **Component Documentation**: Document props, usage examples, and edge cases
3. **Code Comments**: Explain complex logic or non-obvious decisions

## Conclusion

Building a modern Next.js application with Tailwind CSS v4 requires careful planning, a solid understanding of component-based architecture, and attention to detail. By following the practices outlined in this guide, you can create applications that are not only functional but also maintainable, scalable, and enjoyable to work with.

Remember that good organization isn't about following rigid rules, but about creating a structure that makes sense for your specific project and team. The time invested in planning and organization will pay dividends throughout the development process and the life of your application.