## 1xx - Informational
```javascript
100 Continue                // Server received request headers, client should proceed
101 Switching Protocols     // Server switching protocols as requested
102 Processing             // Server processing request, no response available yet
103 Early Hints            // Server sending headers before final response
```

## 2xx - Success
```javascript
200 OK                     // Request successful
201 Created               // Resource created successfully
202 Accepted              // Request accepted but processing not completed
204 No Content            // Request successful, no content to return
206 Partial Content       // Partial GET request successful (used for range requests)
```

## 3xx - Redirection
```javascript
300 Multiple Choices      // Multiple options for resource
301 Moved Permanently     // Resource permanently moved to new URL
302 Found                 // Resource temporarily moved to new URL
303 See Other            // Response found at different URL (use GET)
304 Not Modified         // Resource not modified since last request
307 Temporary Redirect   // Temporary redirect, preserves request method
308 Permanent Redirect   // Permanent redirect, preserves request method
```

## 4xx - Client Errors
```javascript
400 Bad Request          // Server cannot process request due to client error
401 Unauthorized        // Authentication required
403 Forbidden          // Server refuses to fulfill request (even with auth)
404 Not Found          // Resource not found
405 Method Not Allowed // HTTP method not allowed for this resource
408 Request Timeout    // Request took too long
409 Conflict          // Request conflicts with server state
413 Payload Too Large // Request body too large
429 Too Many Requests // Too many requests in given time (rate limiting)
```

## 5xx - Server Errors
```javascript
500 Internal Server Error // Generic server error
501 Not Implemented      // Server doesn't support requested functionality
502 Bad Gateway         // Invalid response from upstream server
503 Service Unavailable // Server temporarily unavailable (overloaded/maintenance)
504 Gateway Timeout    // Upstream server didn't respond in time
```

## Common Usage Examples
```javascript
// Basic success response
response.status(200).json({ message: 'Success' });

// Created new resource
response.status(201).json({ id: 'new-resource-id' });

// No content response
response.status(204).send();

// Bad request with error details
response.status(400).json({
    error: 'Invalid parameters',
    details: 'Email is required'
});

// Unauthorized access
response.status(401).json({
    error: 'Authentication required'
});
```

## Key Points
- `2xx` codes indicate successful requests
- `3xx` codes indicate redirections
- `4xx` codes indicate client-side errors
- `5xx` codes indicate server-side errors
- Use appropriate status codes for better API design
- Include relevant error messages with error status codes
- `201` for resource creation instead of `200`
- `204` when no content needs to be returned
- `404` for not found resources
- `409` for conflicts with existing resources

## Best Practices
- Always send appropriate status codes
- Include meaningful error messages
- Use standard codes over custom ones
- Consider security implications (e.g., 401 vs 403)
- Be consistent across your API
- Document your API's use of status codes

## Common Gotchas
```javascript
// Avoid using 200 for errors
❌ response.status(200).json({ error: 'Something went wrong' });

// Instead use appropriate error code
✅ response.status(400).json({ error: 'Something went wrong' });

// Don't use 404 for validation errors
❌ response.status(404).json({ error: 'Invalid email format' });

// Instead use 400 for validation
✅ response.status(400).json({ error: 'Invalid email format' });
```

## Security Considerations
- Don't expose sensitive information in error messages
- Use 401 for authentication issues
- Use 403 for authorization issues
- Consider rate limiting (429)
- Be careful with redirect codes (3xx)

Related Topics:
- REST API Design
- API Security
- Error Handling
- HTTP Headers
- Request/Response Lifecycle