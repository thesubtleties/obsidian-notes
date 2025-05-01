## 📝 Introduction

Welcome to the world of Model Context Protocol (MCP)! If you've ever worked with web APIs, you're already familiar with the concept of resources - specific endpoints that represent data or functionality. MCP resources work in a similar way, but they're specifically designed for AI models to access and interact with external data and tools.

Think of MCP as a "USB-C for AI" - a standardized way for AI models to connect with the outside world. Just as USB allows different devices to communicate using a common interface, MCP provides a standard protocol for AI models to interact with various resources like databases, APIs, or file systems.

In this guide, we'll explore how to define, expose, and work with MCP resources using the Python SDK. By the end, you'll be able to create your own resources that AI models can seamlessly interact with.

## 🧠 Key Concepts

### What are MCP Resources?

MCP resources are the fundamental building blocks that allow AI models to interact with external data and functionality. Each resource represents a specific capability or data source that you want to make available to an AI model.

Resources in MCP have several key characteristics:

- **URI-based identification**: Each resource has a unique URI (Uniform Resource Identifier) that identifies it
- **Structured interface**: Resources expose a consistent interface for models to interact with
- **Typed data**: Resources define the expected data types for inputs and outputs
- **Stateful or stateless**: Resources can maintain state between interactions or be completely stateless

### Resource URIs

Every MCP resource is identified by a URI that follows this pattern:

```
mcp://{domain}/{path}
```

For example:
- `mcp://example.com/weather` - A weather information resource
- `mcp://myapp.org/database/users` - A resource for accessing user data
- `mcp://tools.ai/calculator` - A calculation tool resource

These URIs serve as unique identifiers that models use to request access to specific resources.

### Resource Types

MCP resources generally fall into a few categories:

1. **Data resources**: Provide access to information (databases, files, APIs)
2. **Tool resources**: Offer functionality (calculators, translators, search engines)
3. **System resources**: Provide access to system capabilities (file system, network)
4. **Composite resources**: Combine multiple resources into a single interface

## 💻 Implementation Example

Let's create a simple MCP resource that provides weather information. We'll implement this step-by-step using the MCP Python SDK.

### Step 1: Install the MCP SDK

```bash
pip install mcp-sdk
```

### Step 2: Define a Weather Resource

```python
from mcp import Resource, ResourceProvider
from typing import Dict, Any, Optional
import json

class WeatherResource(Resource):
    """A resource that provides weather information for a given location."""
    
    def __init__(self):
        # Initialize with a resource URI
        super().__init__(uri="mcp://example.com/weather")
        
        # Sample weather data (in a real app, this would come from an API)
        self.weather_data = {
            "new_york": {"temperature": 72, "condition": "Sunny", "humidity": 45},
            "london": {"temperature": 62, "condition": "Cloudy", "humidity": 80},
            "tokyo": {"temperature": 85, "condition": "Rainy", "humidity": 90},
        }
    
    async def get(self, location: str) -> Dict[str, Any]:
        """Get weather information for a specific location."""
        location = location.lower()
        
        if location in self.weather_data:
            return self.weather_data[location]
        else:
            return {"error": f"Weather data not available for {location}"}
    
    async def list(self) -> Dict[str, Any]:
        """List all available locations."""
        return {"available_locations": list(self.weather_data.keys())}
```

### Step 3: Create a Resource Provider

```python
class MyResourceProvider(ResourceProvider):
    """A provider that exposes our weather resource."""
    
    def __init__(self):
        super().__init__()
        # Register our weather resource
        self.register_resource(WeatherResource())

# Create an instance of our provider
provider = MyResourceProvider()
```

### Step 4: Expose the Resource to a Model

```python
from mcp import MCPSession
import asyncio

async def main():
    # Create an MCP session
    session = MCPSession()
    
    # Register our provider with the session
    session.register_provider(provider)
    
    # Now any model using this session can access our weather resource
    # For example, we can simulate a model requesting weather data:
    weather_resource = session.get_resource("mcp://example.com/weather")
    
    # Get weather for New York
    ny_weather = await weather_resource.get("new_york")
    print(f"New York weather: {json.dumps(ny_weather, indent=2)}")
    
    # List all available locations
    locations = await weather_resource.list()
    print(f"Available locations: {json.dumps(locations, indent=2)}")

# Run the async function
asyncio.run(main())
```

Expected output:
```
New York weather: {
  "temperature": 72,
  "condition": "Sunny",
  "humidity": 45
}
Available locations: {
  "available_locations": [
    "new_york",
    "london",
    "tokyo"
  ]
}
```

### Step 5: Handling Resource Updates

Resources can be dynamic, with data that changes over time. Let's update our weather resource to support updates:

```python
class WeatherResource(Resource):
    # ... previous code ...
    
    async def update(self, location: str, data: Dict[str, Any]) -> Dict[str, Any]:
        """Update weather information for a location."""
        location = location.lower()
        
        if location in self.weather_data:
            # Update existing location data
            for key, value in data.items():
                self.weather_data[location][key] = value
            return {"status": "updated", "location": location}
        else:
            # Add new location
            self.weather_data[location] = data
            return {"status": "added", "location": location}
```

Using the update method:

```python
async def update_example():
    # ... setup code from previous example ...
    
    # Update Tokyo's weather
    update_result = await weather_resource.update("tokyo", {
        "temperature": 78,
        "condition": "Partly Cloudy"
    })
    print(f"Update result: {json.dumps(update_result, indent=2)}")
    
    # Check the updated data
    tokyo_weather = await weather_resource.get("tokyo")
    print(f"Updated Tokyo weather: {json.dumps(tokyo_weather, indent=2)}")

asyncio.run(update_example())
```

Expected output:
```
Update result: {
  "status": "updated",
  "location": "tokyo"
}
Updated Tokyo weather: {
  "temperature": 78,
  "condition": "Partly Cloudy",
  "humidity": 90
}
```

## 🔄 Comparison with Web Development Concepts

If you're coming from a web development background, MCP resources are similar to RESTful API endpoints:

| Web Development Concept | MCP Equivalent | Description |
|-------------------------|----------------|-------------|
| REST API Endpoint | MCP Resource | A specific interface for accessing data or functionality |
| URL/URI | MCP Resource URI | Unique identifier for a resource |
| HTTP Methods (GET, POST) | Resource Methods (get, update) | Actions that can be performed on a resource |
| API Documentation | Resource Schema | Description of how to interact with a resource |
| API Gateway | Resource Provider | Manages and routes requests to appropriate resources |
| Authentication Middleware | MCP Authentication | Controls access to resources |

## 🚀 Best Practices

### 1. Design Clear Resource URIs

- Use descriptive domain and path components
- Follow a consistent naming pattern
- Group related resources under common paths
- Use lowercase letters and hyphens for readability

### 2. Implement Proper Error Handling

```python
async def get(self, location: str) -> Dict[str, Any]:
    try:
        location = location.lower()
        
        if location in self.weather_data:
            return self.weather_data[location]
        else:
            # Return a structured error response
            return {
                "error": {
                    "code": "location_not_found",
                    "message": f"Weather data not available for {location}",
                    "available_locations": list(self.weather_data.keys())
                }
            }
    except Exception as e:
        # Log the error and return a generic error message
        print(f"Error processing request: {str(e)}")
        return {
            "error": {
                "code": "internal_error",
                "message": "An unexpected error occurred"
            }
        }
```

### 3. Document Your Resources

Add clear docstrings and type hints to help others understand how to use your resources:

```python
class WeatherResource(Resource):
    """
    Weather information resource.
    
    This resource provides current weather conditions for various locations.
    
    Methods:
        get(location: str) -> Dict[str, Any]: Get weather for a specific location
        list() -> Dict[str, Any]: List all available locations
        update(location: str, data: Dict[str, Any]) -> Dict[str, Any]: Update weather data
    """
    # ... implementation ...
```

### 4. Implement Resource Validation

Validate inputs to prevent errors and improve security:

```python
async def get(self, location: str) -> Dict[str, Any]:
    # Validate input
    if not location or not isinstance(location, str):
        return {"error": {"code": "invalid_input", "message": "Location must be a non-empty string"}}
    
    location = location.lower()
    # ... rest of the method ...
```

### 5. Use Asynchronous Programming

MCP resources should be implemented using async/await to ensure they don't block the event loop:

```python
# Good - using async/await
async def get_weather_data(location: str) -> Dict[str, Any]:
    # Simulate API call with async
    await asyncio.sleep(0.1)
    return weather_data.get(location, {})

# Bad - blocking call
def get_weather_data_blocking(location: str) -> Dict[str, Any]:
    # This would block the event loop
    time.sleep(0.1)
    return weather_data.get(location, {})
```

## 🔍 Common Issues and Debugging

### Issue 1: Resource Not Found

**Symptom**: The model reports it cannot find your resource.

**Solution**:
- Verify the resource URI is correct and matches exactly
- Ensure the resource provider is properly registered with the session
- Check that the resource is properly registered with the provider

```python
# Debugging code
print(f"Available resources: {session.list_resources()}")
```

### Issue 2: Type Errors

**Symptom**: You get type errors when the model tries to use your resource.

**Solution**:
- Ensure your resource methods have proper type hints
- Validate input data before processing
- Return data in the expected format

### Issue 3: Asynchronous Execution Problems

**Symptom**: Your code hangs or you get "coroutine was never awaited" errors.

**Solution**:
- Make sure all async methods are properly awaited
- Use `asyncio.run()` to run the main async function
- Check for missing `await` keywords in your code

```python
# Incorrect
result = resource.get("location")  # Missing await!

# Correct
result = await resource.get("location")
```

## 📚 Further Learning

### Practice Exercises

1. **Basic Resource**: Create a simple calculator resource that can add, subtract, multiply, and divide numbers.

2. **Data Resource**: Implement a resource that provides access to a JSON file containing book information.

3. **Integration**: Create a resource that integrates with a public API (like a movie database or weather service).

4. **Advanced**: Build a composite resource that combines multiple resources into a single interface.

### Next Steps in Your MCP Journey

- Learn about resource authentication and authorization
- Explore how to create stateful resources that maintain context
- Discover how to optimize resource performance for high-throughput scenarios
- Study how to integrate MCP resources with popular AI models

### Related Topics

- MCP Authentication and Security
- Working with Streaming Resources
- Creating Composite Resources
- Testing MCP Resources
- Deploying MCP Resources in Production

Remember, the key to mastering MCP resources is practice! Try implementing different types of resources and experiment with various patterns to deepen your understanding.