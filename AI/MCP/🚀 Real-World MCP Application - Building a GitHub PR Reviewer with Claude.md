## 📝 Introduction

Welcome to our practical guide on building a GitHub Pull Request (PR) reviewer using the Model Context Protocol (MCP)! In this tutorial, we'll create a complete application that connects Claude AI with GitHub and Notion to automatically review pull requests and provide helpful feedback.

This project brings together many of the MCP concepts we've explored in previous lessons, showing how they work together in a real-world scenario. By the end of this tutorial, you'll have built a tool that can:

1. Monitor a GitHub repository for new pull requests
2. Use Claude to analyze the code changes
3. Generate helpful review comments
4. Store summaries in Notion for team reference

This kind of integration demonstrates the power of MCP as the "USB-C for AI" - allowing Claude to connect with external tools and data sources to create workflows that would be impossible with a standalone AI model.

**Prerequisites:**
- Basic Python knowledge
- GitHub account and basic Git understanding
- Notion account (free tier is fine)
- Anthropic API key
- Completed our previous MCP introduction tutorials or equivalent knowledge

Let's dive in!

## 🧠 Key Concepts

Before we start coding, let's understand the key components of our PR reviewer system:

### MCP Components We'll Use

1. **GitHub Tool**: We'll use MCP to connect Claude with GitHub's API, allowing it to fetch PR details, code diffs, and post comments.

2. **Notion Tool**: We'll use MCP to connect Claude with Notion to store summaries of reviews for team reference.

3. **File Tool**: We'll use this to help Claude analyze larger code files and diffs.

4. **Web Search Tool**: Optionally, we'll allow Claude to search for documentation or best practices when reviewing code.

### System Architecture

Our PR reviewer will follow this workflow:

1. A webhook or scheduled job detects new PRs on GitHub
2. Our Python application fetches the PR details and code changes
3. Claude analyzes the code using MCP tools to access the necessary context
4. Claude generates review comments that are posted back to GitHub
5. A summary of the review is stored in Notion for future reference

Think of this as similar to setting up a CI/CD pipeline, but for AI-powered code reviews!

## 💻 Implementation Example

Let's build our PR reviewer step by step:

### Step 1: Setting Up Your Environment

First, let's set up our Python environment with the necessary packages:

```python
# Create a virtual environment
# python -m venv venv
# source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install required packages
# pip install anthropic python-dotenv PyGithub notion-client
```

Now, create a `.env` file to store your API keys and configuration:

```
ANTHROPIC_API_KEY=your_anthropic_api_key
GITHUB_TOKEN=your_github_personal_access_token
NOTION_TOKEN=your_notion_integration_token
NOTION_DATABASE_ID=your_notion_database_id
GITHUB_REPO_OWNER=your_github_username_or_org
GITHUB_REPO_NAME=your_repository_name
```

### Step 2: Setting Up the Basic Structure

Let's create our main application file:

```python
# pr_reviewer.py
import os
import time
from datetime import datetime
from dotenv import load_dotenv
from github import Github
from notion_client import Client
from anthropic import Anthropic

# Load environment variables
load_dotenv()

# Initialize clients
anthropic_client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
github_client = Github(os.getenv("GITHUB_TOKEN"))
notion_client = Client(auth=os.getenv("NOTION_TOKEN"))

# Get repository
repo = github_client.get_repo(f"{os.getenv('GITHUB_REPO_OWNER')}/{os.getenv('GITHUB_REPO_NAME')}")

# Notion database ID for storing review summaries
NOTION_DATABASE_ID = os.getenv("NOTION_DATABASE_ID")

def main():
    print("🤖 Starting PR Reviewer...")
    # We'll implement this next
    
if __name__ == "__main__":
    main()
```

### Step 3: Fetching Pull Requests from GitHub

Now, let's add the functionality to fetch open pull requests:

```python
def get_open_pull_requests():
    """Fetch all open pull requests from the repository."""
    open_prs = repo.get_pulls(state='open')
    return list(open_prs)

def get_pr_diff(pr):
    """Get the diff content for a pull request."""
    return pr.get_files()

def main():
    print("🤖 Starting PR Reviewer...")
    
    # Get open pull requests
    open_prs = get_open_pull_requests()
    print(f"Found {len(open_prs)} open pull requests")
    
    # Process each PR
    for pr in open_prs:
        print(f"Processing PR #{pr.number}: {pr.title}")
        # We'll implement review logic next
```

### Step 4: Implementing the Claude Review with MCP

Now, let's use Claude with MCP to review the pull requests:

```python
def review_pr_with_claude(pr, files):
    """Use Claude with MCP to review a pull request."""
    
    # Prepare the context for Claude
    pr_description = pr.body or "No description provided."
    
    # Prepare file diffs for Claude to analyze
    file_diffs = []
    for file in files:
        if file.patch:  # Some files might not have a patch (e.g., binary files)
            file_diffs.append(f"File: {file.filename}\nChanges:\n{file.patch}\n")
    
    # Join all diffs, but be mindful of context limits
    all_diffs = "\n".join(file_diffs)
    
    # Create the message for Claude using MCP tools
    message = anthropic_client.messages.create(
        model="claude-3-opus-20240229",
        max_tokens=4000,
        temperature=0.2,
        system="""You are a helpful code reviewer. Your task is to review GitHub pull requests and provide constructive feedback.
        Focus on:
        1. Code quality and best practices
        2. Potential bugs or edge cases
        3. Security concerns
        4. Performance issues
        5. Readability and maintainability
        
        Be specific in your feedback and suggest improvements. Be constructive and helpful, not critical.
        Format your review with markdown, using headings, bullet points, and code blocks as appropriate.
        
        End your review with a summary section that highlights the key points and an overall assessment.
        """,
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": f"""Please review this GitHub pull request:
                        
                        Title: {pr.title}
                        Description: {pr_description}
                        
                        Here are the changes in this PR:
                        """
                    },
                    {
                        "type": "file",
                        "file_id": "file_diff",
                        "title": "PR Diff"
                    }
                ]
            }
        ],
        tools=[
            {
                "name": "file",
                "description": "Tool for reading and writing files",
                "input_schema": {},
                "authentication": {
                    "type": "none"
                }
            },
            {
                "name": "github",
                "description": "Tool for interacting with GitHub repositories",
                "input_schema": {},
                "authentication": {
                    "type": "oauth"
                }
            }
        ],
        tool_resources={
            "file": {
                "file_diff": {
                    "content": all_diffs,
                    "type": "text/plain"
                }
            }
        }
    )
    
    return message.content[0].text

def main():
    print("🤖 Starting PR Reviewer...")
    
    # Get open pull requests
    open_prs = get_open_pull_requests()
    print(f"Found {len(open_prs)} open pull requests")
    
    # Process each PR
    for pr in open_prs:
        print(f"Processing PR #{pr.number}: {pr.title}")
        
        # Get PR files
        files = list(pr.get_files())
        
        # Review PR with Claude
        review = review_pr_with_claude(pr, files)
        
        # Post review as a comment on GitHub
        pr.create_issue_comment(review)
        print(f"✅ Posted review for PR #{pr.number}")
        
        # Store review in Notion (we'll implement this next)
```

### Step 5: Storing Review Summaries in Notion

Let's add the functionality to store review summaries in Notion:

```python
def store_review_in_notion(pr, review):
    """Store a summary of the PR review in Notion."""
    
    # Extract a summary from the review (first 100 characters)
    summary = review[:100] + "..." if len(review) > 100 else review
    
    # Create a new page in the Notion database
    notion_client.pages.create(
        parent={"database_id": NOTION_DATABASE_ID},
        properties={
            "Title": {
                "title": [
                    {
                        "text": {
                            "content": f"PR #{pr.number}: {pr.title}"
                        }
                    }
                ]
            },
            "PR Number": {
                "number": pr.number
            },
            "Author": {
                "rich_text": [
                    {
                        "text": {
                            "content": pr.user.login
                        }
                    }
                ]
            },
            "Review Date": {
                "date": {
                    "start": datetime.now().isoformat()
                }
            },
            "Status": {
                "select": {
                    "name": "Reviewed"
                }
            }
        },
        children=[
            {
                "object": "block",
                "type": "heading_2",
                "heading_2": {
                    "rich_text": [
                        {
                            "type": "text",
                            "text": {
                                "content": "PR Summary"
                            }
                        }
                    ]
                }
            },
            {
                "object": "block",
                "type": "paragraph",
                "paragraph": {
                    "rich_text": [
                        {
                            "type": "text",
                            "text": {
                                "content": pr.body or "No description provided."
                            }
                        }
                    ]
                }
            },
            {
                "object": "block",
                "type": "heading_2",
                "heading_2": {
                    "rich_text": [
                        {
                            "type": "text",
                            "text": {
                                "content": "Review"
                            }
                        }
                    ]
                }
            },
            {
                "object": "block",
                "type": "paragraph",
                "paragraph": {
                    "rich_text": [
                        {
                            "type": "text",
                            "text": {
                                "content": review
                            }
                        }
                    ]
                }
            }
        ]
    )
    
    print(f"📝 Stored review summary in Notion for PR #{pr.number}")

def main():
    print("🤖 Starting PR Reviewer...")
    
    # Get open pull requests
    open_prs = get_open_pull_requests()
    print(f"Found {len(open_prs)} open pull requests")
    
    # Process each PR
    for pr in open_prs:
        print(f"Processing PR #{pr.number}: {pr.title}")
        
        # Get PR files
        files = list(pr.get_files())
        
        # Review PR with Claude
        review = review_pr_with_claude(pr, files)
        
        # Post review as a comment on GitHub
        pr.create_issue_comment(review)
        print(f"✅ Posted review for PR #{pr.number}")
        
        # Store review in Notion
        store_review_in_notion(pr, review)
        
        # Wait a bit before processing the next PR to avoid rate limits
        time.sleep(2)
    
    print("🎉 All PRs processed!")
```

### Step 6: Enhanced MCP Integration with Tool Calling

Let's enhance our PR reviewer by allowing Claude to actively use GitHub and Notion tools through MCP:

```python
def review_pr_with_claude_enhanced(pr, files):
    """Use Claude with MCP to review a pull request with enhanced tool usage."""
    
    # Prepare the context for Claude
    pr_description = pr.body or "No description provided."
    
    # Prepare file diffs for Claude to analyze
    file_diffs = []
    for file in files:
        if file.patch:  # Some files might not have a patch (e.g., binary files)
            file_diffs.append(f"File: {file.filename}\nChanges:\n{file.patch}\n")
    
    # Join all diffs, but be mindful of context limits
    all_diffs = "\n".join(file_diffs)
    
    # Create the message for Claude using MCP tools with tool calling
    message = anthropic_client.messages.create(
        model="claude-3-opus-20240229",
        max_tokens=4000,
        temperature=0.2,
        system="""You are a helpful code reviewer. Your task is to review GitHub pull requests and provide constructive feedback.
        
        You have access to the following tools:
        1. GitHub tool - Use this to get more information about the repository, PR, or to look up specific files
        2. File tool - Use this to analyze code files in detail
        3. Notion tool - Use this to store notes or reference documentation
        
        Focus on:
        4. Code quality and best practices
        5. Potential bugs or edge cases
        6. Security concerns
        7. Performance issues
        8. Readability and maintainability
        
        Be specific in your feedback and suggest improvements. Be constructive and helpful, not critical.
        Format your review with markdown, using headings, bullet points, and code blocks as appropriate.
        
        When appropriate, use the GitHub tool to look up additional context about the repository.
        
        End your review with a summary section that highlights the key points and an overall assessment.
        """,
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": f"""Please review this GitHub pull request:
                        
                        Title: {pr.title}
                        Description: {pr_description}
                        Repository: {os.getenv('GITHUB_REPO_OWNER')}/{os.getenv('GITHUB_REPO_NAME')}
                        PR Number: {pr.number}
                        
                        Here are the changes in this PR:
                        """
                    },
                    {
                        "type": "file",
                        "file_id": "file_diff",
                        "title": "PR Diff"
                    }
                ]
            }
        ],
        tools=[
            {
                "name": "file",
                "description": "Tool for reading and writing files",
                "input_schema": {},
                "authentication": {
                    "type": "none"
                }
            },
            {
                "name": "github",
                "description": "Tool for interacting with GitHub repositories",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "action": {
                            "type": "string",
                            "enum": ["get_file", "get_repo_info", "get_pr_info"]
                        },
                        "repo": {
                            "type": "string"
                        },
                        "file_path": {
                            "type": "string"
                        },
                        "pr_number": {
                            "type": "integer"
                        }
                    },
                    "required": ["action"]
                },
                "authentication": {
                    "type": "oauth"
                }
            },
            {
                "name": "notion",
                "description": "Tool for interacting with Notion",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "action": {
                            "type": "string",
                            "enum": ["create_page", "search"]
                        },
                        "query": {
                            "type": "string"
                        },
                        "content": {
                            "type": "string"
                        }
                    },
                    "required": ["action"]
                },
                "authentication": {
                    "type": "oauth"
                }
            }
        ],
        tool_resources={
            "file": {
                "file_diff": {
                    "content": all_diffs,
                    "type": "text/plain"
                }
            }
        }
    )
    
    return message.content[0].text
```

### Step 7: Putting It All Together

Let's update our main function to use the enhanced review function and add some error handling:

```python
def main():
    print("🤖 Starting PR Reviewer...")
    
    try:
        # Get open pull requests
        open_prs = get_open_pull_requests()
        print(f"Found {len(open_prs)} open pull requests")
        
        # Process each PR
        for pr in open_prs:
            try:
                print(f"Processing PR #{pr.number}: {pr.title}")
                
                # Get PR files
                files = list(pr.get_files())
                
                # Review PR with Claude (enhanced version)
                review = review_pr_with_claude_enhanced(pr, files)
                
                # Post review as a comment on GitHub
                pr.create_issue_comment(review)
                print(f"✅ Posted review for PR #{pr.number}")
                
                # Store review in Notion
                store_review_in_notion(pr, review)
                
                # Wait a bit before processing the next PR to avoid rate limits
                time.sleep(2)
                
            except Exception as e:
                print(f"❌ Error processing PR #{pr.number}: {str(e)}")
                continue
        
        print("🎉 All PRs processed!")
        
    except Exception as e:
        print(f"❌ Error: {str(e)}")

if __name__ == "__main__":
    main()
```

### Step 8: Setting Up Notion Database

Before running our application, we need to set up a Notion database to store our review summaries. Here's how:

1. Create a new integration in Notion (https://www.notion.so/my-integrations)
2. Create a new database in Notion with the following properties:
   - Title (title property)
   - PR Number (number property)
   - Author (text property)
   - Review Date (date property)
   - Status (select property with options like "Reviewed", "Needs Changes", etc.)
3. Share the database with your integration
4. Copy the database ID from the URL (it's the part after the workspace name and before the question mark)

## 🚀 Best Practices

### Error Handling and Resilience

Our example includes basic error handling, but in a production environment, you should:

1. **Add more robust error handling**: Catch specific exceptions and handle them appropriately.
2. **Implement retries**: Use exponential backoff for API calls that might fail.
3. **Add logging**: Use Python's logging module instead of print statements.

```python
import logging

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("pr_reviewer.log"),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger("pr_reviewer")

# Then replace print statements with logger calls
# logger.info("🤖 Starting PR Reviewer...")
# logger.error(f"❌ Error: {str(e)}")
```

### Rate Limiting and Efficiency

To avoid hitting API rate limits:

1. **Check for existing reviews**: Don't review PRs that have already been reviewed.
2. **Implement pagination**: When fetching PRs, use pagination to handle large repositories.
3. **Use webhooks**: Instead of polling, set up GitHub webhooks to trigger reviews when new PRs are created.

```python
def has_bot_reviewed(pr):
    """Check if our bot has already reviewed this PR."""
    comments = pr.get_issue_comments()
    bot_username = github_client.get_user().login
    
    for comment in comments:
        if comment.user.login == bot_username:
            return True
    
    return False
```

### Security Considerations

1. **Store API keys securely**: Use environment variables or a secret management service.
2. **Limit permissions**: Use the principle of least privilege for your GitHub and Notion tokens.
3. **Validate inputs**: Be careful with user-generated content that might be passed to your application.

## 🔍 Common Issues and Debugging

### Context Length Limitations

If you're reviewing large PRs, you might hit Claude's context length limits. To handle this:

1. **Prioritize files**: Focus on the most important files first.
2. **Chunk large diffs**: Split large diffs into multiple reviews.
3. **Filter out non-code files**: Ignore binary files, lock files, etc.

```python
def filter_important_files(files):
    """Filter files to focus on the most important ones."""
    # Ignore certain file types
    ignored_extensions = ['.lock', '.png', '.jpg', '.svg', '.min.js']
    
    important_files = []
    for file in files:
        # Skip binary files and ignored extensions
        if not file.patch:
            continue
        if any(file.filename.endswith(ext) for ext in ignored_extensions):
            continue
        important_files.append(file)
    
    return important_files
```

### API Rate Limits

If you hit rate limits:

1. **GitHub**: Implement exponential backoff and respect the `X-RateLimit-*` headers.
2. **Anthropic**: Add delays between API calls and monitor usage.
3. **Notion**: Batch updates when possible.

### Handling Large Repositories

For large repositories:

1. **Focus on specific directories**: Allow users to specify which directories to review.
2. **Implement caching**: Cache repository information to reduce API calls.
3. **Use incremental reviews**: Only review changes since the last review.

## 📚 Further Learning

### Project Extensions

Here are some ways to extend this project:

1. **Add a web interface**: Create a simple Flask or FastAPI app to manage the PR reviewer.
2. **Implement custom review rules**: Allow teams to define their own review rules.
3. **Add support for multiple repositories**: Scale the reviewer to handle multiple repositories.
4. **Integrate with CI/CD**: Trigger reviews as part of your CI/CD pipeline.
5. **Add support for other AI models**: Compare reviews from different models.

### Practice Exercises

1. **Add language-specific reviews**: Enhance the reviewer to provide specialized feedback based on the programming language.
2. **Implement a review summary dashboard**: Create a web dashboard to visualize review trends.
3. **Add support for code suggestions**: Use Claude to suggest code improvements, not just identify issues.
4. **Implement user feedback**: Allow developers to rate the usefulness of reviews to improve the system over time.

### Related MCP Concepts

This project demonstrates several key MCP concepts:

1. **Tool Orchestration**: Using multiple tools together (GitHub, Notion, File) to create a workflow.
2. **Context Management**: Providing Claude with the right context to make informed decisions.
3. **Tool Authentication**: Handling authentication for external services.
4. **Structured Output**: Formatting reviews in a consistent, useful way.

## 🔗 Connecting to Web Development Concepts

If you're coming from a web development background, here are some familiar concepts that relate to this project:

| Web Development Concept | MCP Equivalent |
|-------------------------|----------------|
| REST API Clients | MCP Tools (GitHub, Notion) |
| Middleware | MCP Tool Processing |
| Environment Variables | API Keys and Configuration |
| Webhooks | Event-driven Architecture |
| Frontend/Backend | Claude (processing) / Tools (data access) |

## 🏁 Conclusion

Congratulations! You've built a complete GitHub PR reviewer using Claude and MCP. This project demonstrates how MCP allows AI models to interact with external tools and data sources, creating powerful workflows that would be impossible with a standalone model.

By connecting Claude to GitHub and Notion, you've created a system that can:
1. Analyze code changes in context
2. Provide helpful, specific feedback
3. Document reviews for future reference

This is just the beginning of what's possible with MCP. As you continue to explore, you'll discover even more powerful ways to connect AI models with the tools and data sources your team already uses.

Remember that the best AI tools augment human capabilities rather than replace them. Your PR reviewer should be seen as a helpful assistant that catches common issues, allowing human reviewers to focus on higher-level concerns like architecture and design.

Happy coding!