## 📝 Introduction

Welcome to the world of Model Context Protocol (MCP)! If you've been working with AI models, you've likely encountered the challenge of connecting them to external data sources, tools, and services. MCP solves this problem by providing a standardized way for AI models to interact with the outside world—think of it as the "USB-C for AI."

In this guide, we'll build a simple but functional MCP server using Python and FastMCP. This server will allow AI models to access external capabilities that you define, such as retrieving information or performing actions. By the end of this tutorial, you'll have a working MCP server that you can extend with your own custom tools.

**Prerequisites:**
- Basic Python knowledge
- Python 3.10+ installed
- Familiarity with web concepts (APIs, HTTP requests)
- A code editor of your choice

## 🧠 Key Concepts

### What is MCP?

The Model Context Protocol (MCP) is a standardized interface that allows AI models to interact with external tools and data sources. If you're familiar with REST APIs, MCP serves a similar purpose but is specifically designed for AI model interactions.

### MCP Server vs. Client

- **MCP Server**: Provides capabilities (tools) that AI models can use
- **MCP Client**: Used by AI models to discover and call the tools on the server

### Core Components

1. **Tools**: Functions that provide specific capabilities (like searching the web, accessing databases, or performing calculations)
2. **Schemas**: Define the structure of data that tools accept and return
3. **Server**: Hosts the tools and makes them available to AI models
4. **Authentication**: Controls access to your MCP server

Think of an MCP server as similar to a REST API server, but with a standardized interface specifically designed for AI models to discover and use your tools.

## 💻 Setting Up Your First MCP Server

### Step 1: Install Required Packages

Let's start by installing the necessary packages:

```bash
pip install fastmcp pydantic
```

`fastmcp` is a Python framework for building MCP servers quickly, similar to how FastAPI helps you build REST APIs. `pydantic` helps with data validation and schema definition.

### Step 2: Create a Basic MCP Server

Let's create a file called `mcp_server.py` with a simple MCP server:

```python
from fastmcp import FastMCP, Tool
from pydantic import BaseModel, Field
import uvicorn

# Initialize the MCP server
app = FastMCP()

# Define a simple tool that returns a greeting
@app.tool()
def hello_world(name: str = "World") -> str:
    """
    Returns a friendly greeting.
    
    Args:
        name: The name to greet
        
    Returns:
        A greeting message
    """
    return f"Hello, {name}! Welcome to MCP!"

# Define a more complex tool with a structured input and output
class WeatherRequest(BaseModel):
    city: str = Field(..., description="The city to get weather for")
    units: str = Field("celsius", description="Temperature units (celsius or fahrenheit)")

class WeatherResponse(BaseModel):
    temperature: float = Field(..., description="Current temperature")
    condition: str = Field(..., description="Weather condition (sunny, cloudy, etc.)")
    humidity: int = Field(..., description="Humidity percentage")

@app.tool()
def get_weather(request: WeatherRequest) -> WeatherResponse:
    """
    Get the current weather for a city.
    
    Args:
        request: The weather request parameters
        
    Returns:
        Current weather information
    """
    # In a real application, you would call a weather API here
    # This is a mock implementation for demonstration
    weather_data = {
        "New York": {"temperature": 22.5, "condition": "Sunny", "humidity": 60},
        "London": {"temperature": 18.0, "condition": "Cloudy", "humidity": 75},
        "Tokyo": {"temperature": 28.0, "condition": "Rainy", "humidity": 80},
    }
    
    # Default data if city not found
    city_data = weather_data.get(request.city, 
                                {"temperature": 20.0, "condition": "Unknown", "humidity": 50})
    
    # Convert temperature if needed
    temp = city_data["temperature"]
    if request.units.lower() == "fahrenheit":
        temp = (temp * 9/5) + 32
    
    return WeatherResponse(
        temperature=temp,
        condition=city_data["condition"],
        humidity=city_data["humidity"]
    )

# Run the server
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Step 3: Run Your MCP Server

Run your server with:

```bash
python mcp_server.py
```

Your MCP server is now running at `http://localhost:8000`!

## 🔍 Understanding the Code

Let's break down what's happening in our MCP server:

### Server Initialization

```python
app = FastMCP()
```

This creates a new FastMCP application, similar to how you'd initialize a Flask or FastAPI app.

### Simple Tool Definition

```python
@app.tool()
def hello_world(name: str = "World") -> str:
    """
    Returns a friendly greeting.
    
    Args:
        name: The name to greet
        
    Returns:
        A greeting message
    """
    return f"Hello, {name}! Welcome to MCP!"
```

This creates a simple tool that:
- Takes a string parameter `name` with a default value of "World"
- Returns a greeting string
- Has a docstring that describes what the tool does (this is important for AI models to understand the tool's purpose)

The `@app.tool()` decorator registers this function as an MCP tool, making it available to AI models.

### Complex Tool with Pydantic Models

```python
class WeatherRequest(BaseModel):
    city: str = Field(..., description="The city to get weather for")
    units: str = Field("celsius", description="Temperature units (celsius or fahrenheit)")

class WeatherResponse(BaseModel):
    temperature: float = Field(..., description="Current temperature")
    condition: str = Field(..., description="Weather condition (sunny, cloudy, etc.)")
    humidity: int = Field(..., description="Humidity percentage")

@app.tool()
def get_weather(request: WeatherRequest) -> WeatherResponse:
    # Implementation...
```

This more complex tool:
- Uses Pydantic models to define structured input and output
- Validates the input data automatically
- Provides clear documentation about the expected parameters and return values
- Returns a structured response

### Running the Server

```python
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

This runs the MCP server using Uvicorn, a fast ASGI server, making your tools available on port 8000.

## 🚀 Exploring Your MCP Server

Once your server is running, you can explore it in several ways:

### 1. View the MCP Manifest

Open your browser and navigate to:
```
http://localhost:8000/.well-known/mcp-manifest
```

This will show you the MCP manifest, which describes all the tools available on your server. This is how AI models discover what your server can do.

### 2. Test Your Tools Directly

You can test your tools by sending HTTP POST requests to their endpoints:

For the `hello_world` tool:
```bash
curl -X POST http://localhost:8000/tools/hello_world -H "Content-Type: application/json" -d '{"name": "Developer"}'
```

For the `get_weather` tool:
```bash
curl -X POST http://localhost:8000/tools/get_weather -H "Content-Type: application/json" -d '{"city": "New York", "units": "celsius"}'
```

## 🔧 Adding Authentication

In a real-world scenario, you'll want to add authentication to your MCP server. Here's how to add a simple API key authentication:

```python
from fastmcp import FastMCP, Tool, Security
from fastmcp.security import APIKeyHeader

# Define security scheme
api_key_header = APIKeyHeader(name="X-API-Key")

# Initialize the MCP server with security
app = FastMCP(security=Security(api_key_header))

# Define a function to validate API keys
@app.api_key_header_auth()
def validate_api_key(api_key: str):
    # In a real application, you would check against a database or environment variable
    valid_keys = ["your-secret-api-key"]
    if api_key not in valid_keys:
        return False
    return True

# Your tools remain the same
@app.tool()
def hello_world(name: str = "World") -> str:
    # ...
```

Now, all requests to your MCP server will require a valid API key in the `X-API-Key` header.

## 🌟 Best Practices

### 1. Tool Design

- **Clear Documentation**: Always include detailed docstrings for your tools
- **Meaningful Names**: Use descriptive names for tools and parameters
- **Input Validation**: Use Pydantic models to validate inputs
- **Error Handling**: Return clear error messages when something goes wrong

### 2. Server Configuration

- **Authentication**: Always add authentication in production
- **HTTPS**: Use HTTPS in production environments
- **Rate Limiting**: Consider adding rate limiting for high-traffic servers
- **Logging**: Implement proper logging for debugging and monitoring

### 3. Code Organization

- **Modular Design**: Split complex tools into separate modules
- **Reusable Components**: Create reusable Pydantic models for common data structures
- **Environment Variables**: Use environment variables for configuration and secrets

## 🐛 Common Issues and Debugging

### Issue: Tool Not Appearing in Manifest

**Possible causes:**
- Missing docstring
- Syntax error in the tool function
- Incorrect return type annotation

**Solution:** Ensure your tool has a proper docstring and correct type annotations.

### Issue: Pydantic Validation Errors

**Possible causes:**
- Sending data that doesn't match the expected schema
- Missing required fields

**Solution:** Check the error message for details and ensure your request data matches the expected schema.

### Issue: Authentication Failures

**Possible causes:**
- Missing or incorrect API key
- Authentication function returning False

**Solution:** Verify your API key and ensure the authentication function is working correctly.

## 🔄 Comparison with Web Development Concepts

| Web Development Concept | MCP Equivalent |
|-------------------------|----------------|
| REST API | MCP Server |
| API Endpoint | MCP Tool |
| OpenAPI/Swagger | MCP Manifest |
| Request/Response Body | Tool Input/Output |
| API Key Auth | MCP Security |
| JSON Schema | Pydantic Models |

If you're familiar with building REST APIs, many of the concepts in MCP will feel familiar, just adapted specifically for AI model interactions.

## 📚 Further Learning

### Practice Exercises

1. **Add a Calculator Tool**: Create a tool that performs basic math operations
2. **Database Integration**: Connect your MCP server to a database to store and retrieve data
3. **External API Integration**: Create a tool that calls an external API (like a real weather API)
4. **File Operations**: Build tools that can read from and write to files

### Next Steps

- Learn how to use your MCP server with AI models
- Explore more advanced FastMCP features like middleware and dependency injection
- Implement more sophisticated authentication mechanisms
- Deploy your MCP server to a cloud provider

### Additional Resources

- [Official MCP Documentation](https://mcp.ai)
- [FastMCP GitHub Repository](https://github.com/fastmcp/fastmcp)
- [Pydantic Documentation](https://docs.pydantic.dev/)

## 🎉 Conclusion

Congratulations! You've built your first MCP server with Python. You now have a foundation for creating powerful tools that AI models can use to interact with external data and services. As you continue your journey with MCP, you'll discover how this standardized protocol can make AI models more capable and useful in real-world applications.

Remember, the key to building great MCP servers is clear documentation, robust error handling, and thoughtful tool design. Happy coding!