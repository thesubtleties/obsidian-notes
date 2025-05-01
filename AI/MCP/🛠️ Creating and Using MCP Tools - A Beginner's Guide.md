## 📌 Overview

This guide introduces you to creating and using Model Context Protocol (MCP) tools - the powerful mechanism that allows AI models to interact with external data, services, and functions. By the end, you'll understand how to build your own tools, integrate them with AI models, and implement best practices for reliable tool execution.

## 📝 Introduction

### What are MCP Tools?

MCP tools are functions that extend an AI model's capabilities by allowing it to interact with external systems and data. Think of them as special abilities you can give to an AI model - like the power to search the web, access a database, or call an API.

### Why are MCP Tools Important?

Without tools, AI models are limited to the knowledge they were trained on and can't access real-time information or perform actions in the world. Tools transform AI models from static knowledge repositories into dynamic assistants that can:

- Retrieve up-to-date information
- Interact with external services
- Perform calculations
- Manipulate data
- Take actions on behalf of users

### How Do MCP Tools Fit into the Ecosystem?

In the MCP ecosystem, tools are the bridge between AI models and the outside world. If MCP is the "USB-C for AI," then tools are the specialized devices you can plug in to extend functionality. They're the mechanism that makes AI models truly useful for real-world applications.

### Real-world Applications

- A customer service AI that can look up order status in your database
- A coding assistant that can run code and return results
- A research assistant that can search the web for current information
- A data analysis AI that can generate charts and visualizations
- A scheduling assistant that can check calendars and book appointments

## 🧠 Key Concepts

### The Tool Decorator Pattern

MCP tools in Python are typically created using the decorator pattern. The `@tool` decorator transforms a regular Python function into an MCP-compatible tool that can be discovered and called by AI models.

### Tool Schemas

Every tool has an input schema that defines what parameters it accepts. This schema serves two purposes:
- It tells the AI model what information it needs to provide when calling the tool
- It validates the inputs to ensure they match expected types and formats

### Synchronous vs. Asynchronous Tools

MCP supports both synchronous and asynchronous tools:
- **Synchronous tools**: Regular functions that execute and return results immediately
- **Asynchronous tools**: Coroutines (using `async/await`) that can perform non-blocking operations

### Tool Registration and Discovery

For an AI model to use your tools, they must be registered with the model's context. The MCP framework handles tool discovery, making your tools available to the model during conversations.

## 💻 Implementation Example

Let's build a set of practical MCP tools step by step, starting with simple examples and progressing to more complex ones.

### Setting Up Your Environment

First, install the necessary packages:

```bash
pip install anthropic mcp-python-sdk pydantic
```

### Basic Tool Example: Weather Lookup

Let's create a simple tool that simulates looking up the weather for a location:

```python
import anthropic
from mcp import tool, MCPContext
from pydantic import BaseModel, Field
from typing import Optional

# Define the input schema for our tool
class WeatherInput(BaseModel):
    location: str = Field(..., description="The city and state/country to get weather for")
    unit: Optional[str] = Field("celsius", description="Temperature unit (celsius or fahrenheit)")

# Create the tool using the decorator pattern
@tool
def get_weather(params: WeatherInput) -> str:
    """
    Get the current weather for a specific location.
    
    This tool returns weather information including temperature and conditions.
    """
    # In a real implementation, this would call a weather API
    # For this example, we'll return mock data
    weather_data = {
        "New York": {"temp": 22, "condition": "Sunny"},
        "London": {"temp": 18, "condition": "Cloudy"},
        "Tokyo": {"temp": 28, "condition": "Rainy"},
        # Add more cities as needed
    }
    
    # Extract the city name from the location
    city = params.location.split(",")[0].strip()
    
    # Get weather data or return a default message
    if city in weather_data:
        data = weather_data[city]
        temp = data["temp"]
        
        # Convert to fahrenheit if requested
        if params.unit.lower() == "fahrenheit":
            temp = (temp * 9/5) + 32
            unit = "°F"
        else:
            unit = "°C"
            
        return f"The weather in {params.location} is {data['condition']} with a temperature of {temp}{unit}."
    else:
        return f"Sorry, I don't have weather data for {params.location}."

# Using the tool with Claude
client = anthropic.Anthropic()

# Create a conversation with our tool
message = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=1000,
    system="You are a helpful assistant that can provide weather information.",
    messages=[
        {"role": "user", "content": "What's the weather like in Tokyo?"}
    ],
    tools=[get_weather],
)

print(message.content)
```

### Asynchronous Tool Example: Database Query

Now let's create an asynchronous tool that simulates querying a database:

```python
import anthropic
from mcp import tool, MCPContext
from pydantic import BaseModel, Field
import asyncio

class QueryInput(BaseModel):
    query: str = Field(..., description="The search query to look up in the database")
    limit: int = Field(5, description="Maximum number of results to return")

@tool
async def search_database(params: QueryInput) -> dict:
    """
    Search the database for information matching the query.
    
    Returns relevant records from the database based on the search query.
    """
    # Simulate an asynchronous database query
    await asyncio.sleep(1)  # Simulate network delay
    
    # Mock database with some sample data
    database = [
        {"id": 1, "title": "Introduction to Python", "category": "Programming"},
        {"id": 2, "title": "Advanced Python Techniques", "category": "Programming"},
        {"id": 3, "title": "Machine Learning Basics", "category": "AI"},
        {"id": 4, "title": "Deep Learning with PyTorch", "category": "AI"},
        {"id": 5, "title": "Web Development with Django", "category": "Web"},
        {"id": 6, "title": "Flask Microservices", "category": "Web"},
        {"id": 7, "title": "Data Analysis with Pandas", "category": "Data Science"},
    ]
    
    # Filter results based on the query (case-insensitive)
    query = params.query.lower()
    results = [
        item for item in database 
        if query in item["title"].lower() or query in item["category"].lower()
    ]
    
    # Apply the limit
    results = results[:params.limit]
    
    return {
        "count": len(results),
        "results": results
    }

# Using the asynchronous tool with Claude requires an async function
async def main():
    client = anthropic.Anthropic()
    
    message = await client.messages.create(
        model="claude-3-opus-20240229",
        max_tokens=1000,
        system="You are a helpful assistant that can search a database.",
        messages=[
            {"role": "user", "content": "Find me some resources about Python"}
        ],
        tools=[search_database],
    )
    
    print(message.content)

# Run the async function
asyncio.run(main())
```

### Complex Tool Example: Multi-step Data Processing

Let's create a more complex tool that processes data in multiple steps:

```python
import anthropic
from mcp import tool, MCPContext
from pydantic import BaseModel, Field
from typing import List, Dict, Any, Optional
import json

class DataProcessingInput(BaseModel):
    data: List[Dict[str, Any]] = Field(..., description="The data to process")
    group_by: str = Field(..., description="The field to group by")
    aggregate: str = Field(..., description="The aggregation method (sum, avg, count)")
    filter_field: Optional[str] = Field(None, description="Optional field to filter on")
    filter_value: Optional[Any] = Field(None, description="Value to filter by")

@tool
def process_data(params: DataProcessingInput) -> Dict[str, Any]:
    """
    Process a dataset by filtering, grouping, and aggregating.
    
    This tool can filter data, group it by a specified field, and perform
    aggregations like sum, average, or count.
    """
    data = params.data
    
    # Step 1: Filter the data if filter parameters are provided
    if params.filter_field and params.filter_value is not None:
        data = [
            item for item in data 
            if item.get(params.filter_field) == params.filter_value
        ]
    
    # Step 2: Group the data by the specified field
    groups = {}
    for item in data:
        key = item.get(params.group_by)
        if key not in groups:
            groups[key] = []
        groups[key].append(item)
    
    # Step 3: Perform the requested aggregation
    results = {}
    for key, group in groups.items():
        if params.aggregate == "count":
            results[key] = len(group)
        elif params.aggregate == "sum":
            # Sum all numeric fields
            sums = {}
            for item in group:
                for field, value in item.items():
                    if isinstance(value, (int, float)) and field != params.group_by:
                        if field not in sums:
                            sums[field] = 0
                        sums[field] += value
            results[key] = sums
        elif params.aggregate == "avg":
            # Average all numeric fields
            sums = {}
            counts = {}
            for item in group:
                for field, value in item.items():
                    if isinstance(value, (int, float)) and field != params.group_by:
                        if field not in sums:
                            sums[field] = 0
                            counts[field] = 0
                        sums[field] += value
                        counts[field] += 1
            
            avgs = {}
            for field, total in sums.items():
                avgs[field] = total / counts[field]
            results[key] = avgs
    
    return {
        "original_count": len(params.data),
        "filtered_count": len(data),
        "groups": len(results),
        "results": results
    }

# Example usage with Claude
client = anthropic.Anthropic()

# Sample data
sample_data = [
    {"product": "Laptop", "category": "Electronics", "price": 1200, "units": 5},
    {"product": "Smartphone", "category": "Electronics", "price": 800, "units": 10},
    {"product": "Desk", "category": "Furniture", "price": 350, "units": 2},
    {"product": "Chair", "category": "Furniture", "price": 150, "units": 8},
    {"product": "Tablet", "category": "Electronics", "price": 500, "units": 7}
]

# Create a conversation with our tool
message = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=1000,
    system="You are a helpful data analysis assistant.",
    messages=[
        {"role": "user", "content": f"""
        I have this product data:
        {json.dumps(sample_data, indent=2)}
        
        Can you group it by category and calculate the average price for each category?
        """}
    ],
    tools=[process_data],
)

print(message.content)
```

### Tool with File Handling

Let's create a tool that can handle file operations:

```python
import anthropic
from mcp import tool, MCPContext
from pydantic import BaseModel, Field
import os
import json
from typing import Optional, List, Dict, Any

class FileOperationInput(BaseModel):
    operation: str = Field(..., description="Operation to perform (read, write, list)")
    path: str = Field(..., description="File or directory path")
    content: Optional[str] = Field(None, description="Content to write (for write operation)")
    format: Optional[str] = Field("text", description="Format for reading/writing (text or json)")

@tool
def file_operation(params: FileOperationInput) -> Dict[str, Any]:
    """
    Perform file operations like reading, writing, or listing files.
    
    This tool can read file contents, write data to files, or list directory contents.
    """
    result = {"success": False, "message": "", "data": None}
    
    try:
        # List directory contents
        if params.operation == "list":
            if os.path.isdir(params.path):
                files = os.listdir(params.path)
                result["data"] = files
                result["success"] = True
                result["message"] = f"Listed {len(files)} items in directory"
            else:
                result["message"] = "Path is not a directory"
        
        # Read file contents
        elif params.operation == "read":
            if os.path.isfile(params.path):
                mode = "r"
                if params.format == "json":
                    with open(params.path, mode) as file:
                        result["data"] = json.load(file)
                else:
                    with open(params.path, mode) as file:
                        result["data"] = file.read()
                result["success"] = True
                result["message"] = "File read successfully"
            else:
                result["message"] = "File does not exist"
        
        # Write content to file
        elif params.operation == "write":
            if params.content is not None:
                mode = "w"
                directory = os.path.dirname(params.path)
                if directory and not os.path.exists(directory):
                    os.makedirs(directory)
                    
                if params.format == "json" and isinstance(params.content, str):
                    # If content is provided as a string but format is JSON,
                    # try to parse it as JSON first
                    try:
                        data = json.loads(params.content)
                        with open(params.path, mode) as file:
                            json.dump(data, file, indent=2)
                    except json.JSONDecodeError:
                        # If parsing fails, write as plain text
                        with open(params.path, mode) as file:
                            file.write(params.content)
                else:
                    with open(params.path, mode) as file:
                        file.write(params.content)
                        
                result["success"] = True
                result["message"] = "File written successfully"
            else:
                result["message"] = "No content provided for writing"
        else:
            result["message"] = f"Unsupported operation: {params.operation}"
            
    except Exception as e:
        result["message"] = f"Error: {str(e)}"
    
    return result

# Example usage with Claude
client = anthropic.Anthropic()

# Create a conversation with our tool
message = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=1000,
    system="You are a helpful assistant that can perform file operations.",
    messages=[
        {"role": "user", "content": "Can you write a simple JSON file with a list of fruits to 'fruits.json'?"}
    ],
    tools=[file_operation],
)

print(message.content)
```

## 🚀 Best Practices

### Input Validation

Always use Pydantic models to define your tool inputs. This provides several benefits:

1. **Automatic validation**: Pydantic validates that inputs match the expected types
2. **Self-documentation**: The schema is used to generate documentation for the AI model
3. **Default values**: You can specify default values for optional parameters
4. **Clear error messages**: When validation fails, you get helpful error messages

```python
class SearchInput(BaseModel):
    query: str = Field(..., description="The search query")
    max_results: int = Field(5, description="Maximum number of results", ge=1, le=50)
    include_images: bool = Field(False, description="Whether to include images in results")
```

### Descriptive Documentation

Always include clear docstrings for your tools. The AI model uses these to understand when and how to use your tools:

```python
@tool
def search_products(params: SearchInput) -> dict:
    """
    Search for products in the catalog.
    
    This tool searches the product database and returns matching items based on
    the search query. Results can be limited and may include product images.
    """
    # Implementation here
```

### Error Handling

Implement robust error handling in your tools to prevent crashes and provide helpful feedback:

```python
@tool
def calculate_statistics(params: StatsInput) -> dict:
    """Calculate statistical measures for a dataset."""
    try:
        # Main implementation
        return {"mean": mean_value, "median": median_value}
    except ValueError as e:
        # Handle specific errors
        return {"error": f"Invalid data format: {str(e)}"}
    except Exception as e:
        # Catch-all for unexpected errors
        return {"error": f"An unexpected error occurred: {str(e)}"}
```

### Tool Organization

For applications with many tools, organize them into logical groups:

```python
# Database tools
@tool
def query_database(params: QueryInput) -> dict:
    """Query the database."""
    # Implementation

@tool
def update_record(params: UpdateInput) -> dict:
    """Update a database record."""
    # Implementation

# Analytics tools
@tool
def generate_report(params: ReportInput) -> dict:
    """Generate an analytics report."""
    # Implementation
```

### Performance Considerations

1. **Use async for I/O-bound operations**: Network requests, database queries, and file operations benefit from async implementation
2. **Keep tools focused**: Each tool should do one thing well
3. **Cache expensive results**: If a tool performs expensive operations, consider caching results
4. **Monitor execution time**: Log how long tools take to execute to identify bottlenecks

```python
import time
import functools

def log_execution_time(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Tool {func.__name__} executed in {end_time - start_time:.2f} seconds")
        return result
    return wrapper

@tool
@log_execution_time
def expensive_operation(params: OperationInput) -> dict:
    """Perform an expensive operation."""
    # Implementation
```

## 🔍 Common Issues and Debugging

### Tool Not Being Called

If the AI model isn't using your tool when expected:

1. **Check the docstring**: Make sure it clearly describes when the tool should be used
2. **Verify the input schema**: Complex or unclear schemas can confuse the model
3. **Review the conversation**: The model might not understand that the tool is needed

### Input Validation Errors

When your tool receives invalid inputs:

1. **Check the schema definitions**: Make sure field types and constraints are correct
2. **Add more descriptive field documentation**: Help the model understand what to provide
3. **Use appropriate validators**: Pydantic offers custom validators for complex validation

```python
from pydantic import BaseModel, Field, validator

class DateRangeInput(BaseModel):
    start_date: str = Field(..., description="Start date in YYYY-MM-DD format")
    end_date: str = Field(..., description="End date in YYYY-MM-DD format")
    
    @validator('start_date', 'end_date')
    def validate_date_format(cls, v):
        try:
            # Attempt to parse the date
            datetime.strptime(v, '%Y-%m-%d')
            return v
        except ValueError:
            raise ValueError(f"Date must be in YYYY-MM-DD format, got: {v}")
    
    @validator('end_date')
    def validate_date_range(cls, v, values):
        if 'start_date' in values and v < values['start_date']:
            raise ValueError("End date must be after start date")
        return v
```

### Debugging Tool Execution

When troubleshooting tool behavior:

1. **Add logging**: Log inputs, outputs, and intermediate steps
2. **Test tools independently**: Before using with the AI, test your tools directly
3. **Use try/except blocks**: Catch and handle specific errors

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@tool
def search_database(params: SearchInput) -> dict:
    """Search the database."""
    logger.info(f"Received search request with params: {params}")
    
    try:
        # Implementation
        results = perform_search(params.query, params.max_results)
        logger.info(f"Search returned {len(results)} results")
        return {"results": results}
    except Exception as e:
        logger.error(f"Search failed with error: {str(e)}")
        raise
```

## 📚 Further Learning

### Practice Exercises

1. **Basic Tool Creation**: Create a calculator tool that can perform basic arithmetic operations
2. **API Integration**: Build a tool that fetches data from a public API (like weather, news, or stock prices)
3. **Database Tool**: Create a tool that interacts with a database (SQLite is good for practice)
4. **File Processing Tool**: Build a tool that can read, write, and analyze text files
5. **Multi-step Tool**: Create a tool that performs a sequence of operations and returns a processed result

### Advanced Topics to Explore

- **Tool Composition**: Creating complex workflows by combining multiple tools
- **Stateful Tools**: Managing state between tool calls
- **Tool Authentication**: Securely handling credentials for external services
- **Rate Limiting**: Implementing rate limiting for tools that call external APIs
- **Streaming Responses**: Creating tools that return data incrementally

### Related MCP Concepts

- **Tool Chaining**: How models can use multiple tools in sequence to solve complex problems
- **Context Management**: Managing the context size when tools return large amounts of data
- **Prompt Engineering**: Crafting effective prompts that guide tool usage
- **Model Grounding**: Using tools to ground model responses in factual data

## 🔄 Comparison to Web Development Concepts

For bootcamp graduates familiar with web development, here are some helpful comparisons:

| MCP Concept | Web Development Equivalent |
|-------------|----------------------------|
| MCP Tools | API Endpoints or Microservices |
| Tool Decorator | Express/Flask Route Handlers |
| Input Schema | Request Validation Middleware |
| Tool Documentation | API Documentation (Swagger/OpenAPI) |
| Async Tools | Async Route Handlers in Express/Node.js |
| Tool Registration | Registering Routes with a Router |
| Tool Execution | API Request/Response Cycle |

Just as you would define routes in a web application to handle specific requests, you define tools in MCP to handle specific capabilities the AI model might need. The key difference is that instead of human users directly calling your endpoints, the AI model decides when and how to use your tools based on the conversation context.

---

By following this guide, you've learned how to create and use MCP tools to extend AI capabilities. You've seen examples ranging from simple synchronous tools to complex asynchronous data processing, along with best practices for implementation. As you continue your journey with MCP, experiment with different tool types and integrations to build increasingly powerful AI applications!