## Basic Form Setup
```javascript
// src/components/ProductForm.jsx
import { Form, useActionData, useNavigation } from 'react-router-dom';

function ProductForm() {
  const actionData = useActionData(); // Get action results/errors
  const navigation = useNavigation();
  const isSubmitting = navigation.state === "submitting";
  
  return (
    <Form method="post">
      <div>
        <label htmlFor="name">Product Name</label>
        <input 
          id="name"
          name="name" 
          type="text"
          aria-invalid={actionData?.errors?.name ? true : undefined}
        />
        {actionData?.errors?.name && (
          <span className="error">{actionData.errors.name}</span>
        )}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Saving..." : "Save Product"}
      </button>
    </Form>
  );
}
```
The `Form` component from React Router replaces traditional HTML forms. It automatically handles form submissions through route actions, provides loading states via `useNavigation`, and gives access to submission results through `useActionData`. This eliminates the need for manual form handling and provides a more integrated approach to form management.

## Form Actions
```javascript
// src/actions/productActions.js
export async function createProductAction({ request }) {
  // Get form data
  const formData = await request.formData();
  const data = Object.fromEntries(formData);
  
  // Validate
  const errors = validateProduct(data);
  if (Object.keys(errors).length > 0) {
    return { errors };
  }
  
  try {
    const response = await fetch('/api/products', {
      method: 'POST',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error('Failed to create product');
    }

    // Redirect on success
    return redirect('/products');
  } catch (error) {
    return { errors: { _form: 'Failed to create product' } };
  }
}
```
Actions are server-side functions that handle form submissions. They receive the form data, process it, and return either success data, validation errors, or redirect to another route. This provides a clean separation between form UI and form processing logic.

## File Upload Form
```javascript
// src/components/ProductForm.jsx
function ProductForm() {
  return (
    <Form method="post" encType="multipart/form-data">
      <input type="text" name="name" />
      <input type="file" name="image" accept="image/*" />
      <button type="submit">Create Product</button>
    </Form>
  );
}

// src/actions/productActions.js
export async function productAction({ request }) {
  const formData = await request.formData();
  
  // Handle file upload
  const file = formData.get('image');
  if (file) {
    const imageUrl = await uploadImage(file);
    formData.set('imageUrl', imageUrl);
    formData.delete('image');
  }
  
  // Process the rest of the form...
}
```
File uploads require special handling. The form needs the `encType` attribute, and the action needs to process the file data differently from regular form fields. This example shows how to handle file uploads within the React Router form system.

## Dynamic Form with Validation
```javascript
function DynamicForm() {
  const actionData = useActionData();
  const navigation = useNavigation();
  const [fields, setFields] = useState([{ id: 1 }]);
  
  return (
    <Form method="post">
      {fields.map((field, index) => (
        <div key={field.id}>
          <input 
            name={`variant[${index}]`}
            defaultValue={actionData?.fields?.[index]}
          />
          {actionData?.errors?.[`variant[${index}]`] && (
            <span className="error">
              {actionData.errors[`variant[${index}]`]}
            </span>
          )}
        </div>
      ))}
      
      <button 
        type="button" 
        onClick={() => setFields([...fields, { id: Date.now() }])}
      >
        Add Variant
      </button>
      
      <button type="submit">Save</button>
    </Form>
  );
}
```
Dynamic forms allow users to add or remove form fields. This example shows how to manage dynamic fields while maintaining proper validation and error handling. The form can grow or shrink based on user interaction while still working with React Router's form handling system.

## Form Validation Patterns

### Client-Side Validation
```javascript
// src/components/ValidatedForm.jsx
import { useFormValidation } from '../hooks/useFormValidation';

function ValidatedForm() {
  const { errors, validateField } = useFormValidation();
  
  return (
    <Form method="post" onSubmit={e => {
      if (!validateField('email')) {
        e.preventDefault();
      }
    }}>
      <input 
        name="email" 
        type="email"
        onChange={e => validateField('email', e.target.value)}
      />
      {errors.email && <span className="error">{errors.email}</span>}
      <button type="submit">Submit</button>
    </Form>
  );
}
```
Client-side validation provides immediate feedback to users before the form is submitted. This example shows how to integrate real-time validation while still maintaining compatibility with React Router's form handling. The form can prevent submission if validation fails.

### Server-Side Validation
```javascript
// src/utils/validation.js
export function validateProduct(data) {
  const errors = {};
  
  if (!data.name?.trim()) {
    errors.name = 'Name is required';
  }
  
  if (data.price && isNaN(data.price)) {
    errors.price = 'Price must be a number';
  }
  
  return errors;
}

// src/actions/productActions.js
export async function productAction({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);
  
  // Validate
  const errors = validateProduct(data);
  if (Object.keys(errors).length > 0) {
    return { errors, fields: data }; // Return fields for re-populating form
  }
  
  // Process valid data...
}
```
Server-side validation is crucial for data integrity. This pattern shows how to validate form data in the action and return validation errors to the form. It also demonstrates how to preserve form data when validation fails, providing a better user experience.

## Form State Management
```javascript
// src/components/StateAwareForm.jsx
function StateAwareForm() {
  const navigation = useNavigation();
  const actionData = useActionData();
  const isSubmitting = navigation.state === "submitting";
  const isSuccess = actionData?.success;
  
  return (
    <div>
      {isSuccess && (
        <div className="success-message">Product saved!</div>
      )}
      
      <Form method="post">
        {/* Form fields */}
        
        <button 
          type="submit" 
          disabled={isSubmitting}
          className={isSubmitting ? 'loading' : ''}
        >
          {isSubmitting ? 'Saving...' : 'Save'}
        </button>
      </Form>
      
      {actionData?.errors?._form && (
        <div className="error-message">
          {actionData.errors._form}
        </div>
      )}
    </div>
  );
}
```
Managing form state is crucial for good UX. This example shows how to handle different form states (submitting, success, error) using React Router's built-in hooks. It provides visual feedback for form submission status and displays appropriate messages to users.

## TypeScript Support
```typescript
// src/types/forms.ts
interface ProductFormData {
  name: string;
  price: number;
  description?: string;
}

// src/actions/productActions.ts
export async function productAction({ 
  request 
}: ActionFunctionArgs): Promise<ProductFormData | { errors: Record<string, string> }> {
  const formData = await request.formData();
  const data = Object.fromEntries(formData) as ProductFormData;
  
  // Type-safe validation...
  
  return data;
}
```
TypeScript support provides type safety for form data and actions. This pattern shows how to type your form data and actions, ensuring type safety throughout the form handling process and catching potential errors at compile time.

## Common Patterns and Best Practices

### Form Reset After Submit
```javascript
function ResetForm() {
  const navigation = useNavigation();
  const formRef = useRef<HTMLFormElement>(null);
  
  useEffect(() => {
    if (navigation.state === "idle" && formRef.current) {
      formRef.current.reset();
    }
  }, [navigation.state]);
  
  return <Form ref={formRef} method="post">{/* ... */}</Form>;
}
```
Sometimes you want to reset a form after successful submission. This pattern shows how to automatically reset form fields when the submission is complete, providing a clean slate for the next submission.

### Optimistic UI Updates
```javascript
function OptimisticForm() {
  const [items, setItems] = useState([]);
  
  return (
    <Form 
      method="post" 
      onSubmit={(e) => {
        const formData = new FormData(e.currentTarget);
        const newItem = formData.get('item');
        // Update UI immediately
        setItems(prev => [...prev, newItem]);
      }}
    >
      {/* Form fields */}
    </Form>
  );
}
```
Optimistic UI updates improve perceived performance by updating the UI before the server responds. This pattern shows how to implement optimistic updates while still using React Router's form handling system.

## Common Gotchas
```javascript
// Form Data Handling
// ❌ Incorrect
export async function badAction({ request }) {
  const data = await request.json(); // Won't work with Form
}

// ✅ Correct
export async function goodAction({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);
}

// Error Handling
// ❌ Incorrect
export async function badAction() {
  try {
    await saveData();
  } catch (error) {
    return error; // Raw error object
  }
}

// ✅ Correct
export async function goodAction() {
  try {
    await saveData();
    return { success: true };
  } catch (error) {
    return { 
      errors: { 
        _form: 'Failed to save data'
      } 
    };
  }
}
```
These examples show common mistakes and their corrections when handling forms in React Router. The key is to properly handle form data using formData() instead of json(), and to structure error responses in a way that the form can properly display them.

