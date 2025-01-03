
```javascript
const API_BASE_URL = '/api';

export async function apiCall(endpoint, method = 'GET', body = null) {
  const options = {
    method,
    headers: {
      'Content-Type': 'application/json',
    },
  };

  if (body) {
    options.body = JSON.stringify(body);
  }

  const response = await fetch(`${API_BASE_URL}${endpoint}`, options);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'An error occurred');
  }

  return response.json();
}
```

This function serves as a wrapper around the native `fetch` API. Here's what it does:

1. `API_BASE_URL`: 
   - Defines the base URL for all API calls. In this case, it's set to '/api', which is common when your frontend and backend are served from the same domain.

2. Function Parameters:
   - `endpoint`: The specific API endpoint you're calling (e.g., '/users', '/login').
   - `method`: The HTTP method (GET, POST, PUT, DELETE, etc.). Defaults to 'GET'.
   - `body`: The data to send with the request (for POST, PUT, etc.). Optional.

3. Request Configuration:
   - Creates an `options` object to configure the fetch request.
   - Sets the `Content-Type` header to 'application/json' for all requests.

4. Request Body:
   - If a `body` is provided, it's stringified and added to the options.

5. Making the Request:
   - Uses `fetch` to make the API call, combining the base URL and the specific endpoint.

6. Error Handling:
   - Checks if the response is not OK (status outside 200-299 range).
   - If there's an error, it attempts to parse the error message from the response.
   - Throws an error with the parsed message or a default message.

7. Response Parsing:
   - If the response is OK, it parses and returns the JSON response.

Benefits of this utility:

1. Centralization: All API calls go through this function, making it easier to manage and modify global API behavior.
2. DRY (Don't Repeat Yourself): Reduces repetitive code in components and actions.
3. Consistent Error Handling: Provides a standard way to handle API errors.
4. Easy Configuration: Makes it simple to add headers, change the base URL, or modify request options globally.

Using this utility in your Redux actions or components simplifies API interactions. For example:

```javascript
import { apiCall } from '../utils/api';

export const fetchUsers = () => async (dispatch) => {
  try {
    const users = await apiCall('/users');
    dispatch({ type: 'SET_USERS', payload: users });
  } catch (error) {
    dispatch({ type: 'SET_ERROR', payload: error.message });
  }
};
```

This approach keeps your API interaction code clean and consistent throughout your application.