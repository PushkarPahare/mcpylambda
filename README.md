# mcpylambda 🎼

**Transform your APIs into AI agent tools with zero hassle**

Mcpylambda is a Python framework that makes it incredibly easy to create Model Context Protocol (MCP) servers in AWS Lambda. Turn your existing APIs into tools that AI agents can discover and use automatically.

[![Python 3.7+](https://img.shields.io/badge/python-3.7+-blue.svg)](https://www.python.org/downloads/)
[![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-orange.svg)](https://aws.amazon.com/lambda/)
[![MCP Protocol](https://img.shields.io/badge/MCP-2024--11--05-green.svg)](https://modelcontextprotocol.io/)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen.svg)](#)

## 🚀 **Why Mcpylambda?**

```python
# Before: Complex MCP server setup, manual schema generation, protocol handling...

# After: Just add decorators to your functions
@framework.tool(description="Add two numbers")
def add(a: float, b: float, client: CalculatorClient) -> dict:
    return {"result": client.add(a, b)}

# That's it! Your API is now an AI agent tool 🎉
```

### **The Problem**
- Setting up MCP servers is complex and time-consuming
- Manual JSON schema generation for every tool
- Boilerplate code for client management and dependency injection
- Lambda deployment complexity for serverless MCP servers

### **The Solution**
Mcpylambda handles all the complexity so you can focus on your business logic:
- ✅ **Decorator-based API** - Just add `@framework.tool()`
- ✅ **Automatic schema generation** - From Python type hints
- ✅ **Client dependency injection** - Type-hint based, automatic
- ✅ **Lambda optimized** - Built specifically for AWS Lambda
- ✅ **Zero external dependencies** - Only Python standard library
- ✅ **Copy-paste ready** - Single file, no installation needed

## 📦 **Installation**

### Option 1: Copy & Paste (Recommended for Lambda)
```bash
# Download the single file
curl -O https://raw.githubusercontent.com/your-repo/Mcpylambda/main/Mcpylambda.py

# Copy into your Lambda function - that's it!
```

### Option 2: Package Installation
```bash
pip install Mcpylambda-mcp
```

## ⚡ **Quick Start**

### 1. **Create Your Client**
```python
from Mcpylambda import MCPClient, framework

class MyAPIClient(MCPClient):
    def authenticate(self) -> bool:
        return os.environ.get('API_KEY') is not None
    
    def get_user(self, user_id: int) -> dict:
        # Your existing API logic
        return requests.get(f"https://api.example.com/users/{user_id}").json()
    
    def create_user(self, name: str, email: str) -> dict:
        # Your existing API logic  
        data = {"name": name, "email": email}
        return requests.post("https://api.example.com/users", json=data).json()
```

### 2. **Create Your Tools**
```python
@framework.tool(description="Get user information")
def get_user(user_id: int, client: MyAPIClient) -> dict:
    """Get user details by ID."""
    return client.get_user(user_id)

@framework.tool(description="Create a new user")
def create_user(name: str, email: str, client: MyAPIClient) -> dict:
    """Create a new user account."""
    return client.create_user(name, email)
```

### 3. **Deploy to Lambda**
```python
def lambda_handler(event, context):
    return framework.handle_lambda_event(event, context)
```

### 4. **Connect Your Agent**
```json
{
  "mcpServers": {
    "my-api": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-stdio", "https://your-api-gateway-url"]
    }
  }
}
```

**That's it!** Your AI agent can now discover and use your API tools automatically.

## 🎯 **Key Features**

### **🔧 Automatic Tool Discovery**
```python
# Each decorated function becomes an MCP tool
@framework.tool(description="Calculate fibonacci number")
def fibonacci(n: int, client: MathClient) -> int:
    return client.calculate_fibonacci(n)

# Agent automatically discovers:
# - Tool name: "fibonacci"  
# - Description: "Calculate fibonacci number"
# - Parameters: {"n": {"type": "integer"}}
# - Required fields: ["n"]
```

### **🧠 Smart Schema Generation**
```python
def complex_operation(
    name: str,
    count: int, 
    enabled: bool = True,
    tags: List[str] = None
) -> dict:
    pass

# Automatically generates:
# {
#   "type": "object",
#   "properties": {
#     "name": {"type": "string"},
#     "count": {"type": "integer"}, 
#     "enabled": {"type": "boolean"},
#     "tags": {"type": "array", "items": {"type": "string"}}
#   },
#   "required": ["name", "count"]
# }
```

### **💉 Client Dependency Injection**
```python
class DatabaseClient(MCPClient): ...
class EmailClient(MCPClient): ...

@framework.tool()
def send_user_report(user_id: int, db: DatabaseClient, email: EmailClient):
    # Both clients automatically injected based on type hints
    user = db.get_user(user_id)
    report = db.generate_report(user_id)
    email.send_report(user.email, report)
```

### **🌐 Multiple MCP Features**
```python
# Tools - Functions agents can call
@framework.tool(description="Process payment")
def process_payment(amount: float, client: PaymentClient) -> dict: ...

# Resources - Data sources agents can access  
@framework.resource("myapi://user/{user_id}")
def get_user_resource(user_id: str, client: UserClient) -> str: ...

# Prompts - Template generators
@framework.prompt(description="Generate user analysis")
def analyze_user(user_id: int, focus: str = "behavior") -> str: ...
```

## 📚 **Examples**

### **Calculator Service**
```python
class CalculatorClient(MCPClient):
    def authenticate(self) -> bool:
        return True  # No auth needed
    
    def add(self, a: float, b: float) -> float:
        return a + b

@framework.tool(description="Add two numbers")
def add(a: float, b: float, client: CalculatorClient) -> dict:
    result = client.add(a, b)
    return {
        "operation": "addition",
        "inputs": [a, b],
        "result": result,
        "expression": f"{a} + {b} = {result}"
    }
```

### **User Management API**
```python
class UserAPIClient(MCPClient):
    def authenticate(self) -> bool:
        return os.environ.get('USER_API_KEY') is not None
    
    def get_user(self, user_id: int) -> dict:
        headers = {'Authorization': f'Bearer {os.environ["USER_API_KEY"]}'}
        response = requests.get(f'https://api.users.com/v1/users/{user_id}', headers=headers)
        return response.json()

@framework.tool(description="Get user profile")
def get_user_profile(user_id: int, client: UserAPIClient) -> dict:
    return client.get_user(user_id)

@framework.resource("users://profile/{user_id}")
def user_profile_resource(user_id: str, client: UserAPIClient) -> str:
    user = client.get_user(int(user_id))
    return f"User: {user['name']} ({user['email']})\nJoined: {user['created_at']}"
```

### **Multi-Client Architecture**
```python
class ReadOnlyDBClient(MCPClient):
    def get_user(self, user_id: int) -> dict: ...
    def search_users(self, query: str) -> List[dict]: ...

class WriteDBClient(MCPClient):  
    def create_user(self, data: dict) -> dict: ...
    def update_user(self, user_id: int, data: dict) -> dict: ...

# Read operations use read-only client
@framework.tool(description="Search for users")
def search_users(query: str, client: ReadOnlyDBClient) -> List[dict]:
    return client.search_users(query)

# Write operations use write client  
@framework.tool(description="Create new user")
def create_user(name: str, email: str, client: WriteDBClient) -> dict:
    return client.create_user({"name": name, "email": email})
```

## 🏗️ **Architecture**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   AI Agent      │    │   AWS Lambda     │    │   Your APIs     │
│  (Claude, etc.) │◄──►│   + Mcpylambda    │◄──►│  (Any Service)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                        │                        │
        │                        │                        │
    MCP Protocol          Framework handles:         Your business
    JSON-RPC 2.0          • Protocol compliance      logic & APIs
    Tool discovery        • Schema generation        
    Tool execution        • Client injection         
                         • Error handling           
```

### **How It Works**

1. **Agent Discovery**: Agent calls `tools/list` → Framework returns all decorated functions
2. **Schema Generation**: Framework generates JSON schemas from Python type hints  
3. **Tool Execution**: Agent calls `tools/call` → Framework injects clients and calls your function
4. **Client Management**: Framework manages client instances with singleton pattern for Lambda efficiency
5. **Response Formatting**: Framework converts your return values to MCP-compliant responses

## 🚀 **Deployment**

### **AWS Lambda + API Gateway**
```yaml
# serverless.yml
service: my-mcp-server

provider:
  name: aws
  runtime: python3.9
  environment:
    API_KEY: ${env:API_KEY}

functions:
  mcp:
    handler: my_mcp_server.lambda_handler
    events:
      - http:
          path: mcp
          method: post
          cors: true
```

### **AWS CDK**
```python
from aws_cdk import aws_lambda as _lambda

lambda_function = _lambda.Function(
    self, "MCPServer",
    runtime=_lambda.Runtime.PYTHON_3_9,
    handler="my_mcp_server.lambda_handler", 
    code=_lambda.Code.from_asset("src"),
    environment={
        "API_KEY": api_key_secret.secret_value.to_string()
    }
)
```

### **Direct Lambda Invocation**
```python
# Test directly in Lambda console
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}
```

## 🧪 **Testing**

### **Local Testing**
```python
# Run your MCP server directly
python my_mcp_server.py

# Output:
# Available tools:
# - get_user: Get user information  
# - create_user: Create a new user
# - search_users: Search for users
```

### **MCP Protocol Testing**
```python
# Test tools discovery
test_event = {
    "jsonrpc": "2.0", 
    "method": "tools/list",
    "id": 1
}

result = lambda_handler(test_event, None)
print(f"Found {len(result['result']['tools'])} tools")

# Test tool execution
call_event = {
    "jsonrpc": "2.0",
    "method": "tools/call", 
    "params": {
        "name": "get_user",
        "arguments": {"user_id": 123}
    },
    "id": 2
}

result = lambda_handler(call_event, None)
```

### **Automated Testing**
```bash
# Use the included test script
python test_mcp_server.py

# Comprehensive test output:
# ✅ MCP initialization successful
# ✅ Found 5 tools with proper schemas  
# ✅ Tool calls execute correctly
# ✅ Resources and prompts discovered
```

## 🔧 **Advanced Usage**

### **Custom Response Types**
```python
from Mcpylambda import MCPToolResult, MCPContent, MCPContentType

@framework.tool()
def advanced_analysis(query: str) -> MCPToolResult:
    return MCPToolResult(
        content=[
            MCPContent(
                type=MCPContentType.TEXT,
                text="Analysis complete"
            ),
            MCPContent(
                type=MCPContentType.IMAGE, 
                data=chart_image_bytes,
                mimeType="image/png"
            )
        ],
        isError=False
    )
```

### **Error Handling**
```python
@framework.tool()
def risky_operation(data: str, client: APIClient) -> dict:
    try:
        return client.process_data(data)
    except APIException as e:
        return {
            "error": str(e),
            "success": False,
            "retry_after": 60
        }
```

### **Resource Templates**
```python
@framework.resource("myapi://report/{report_id}/section/{section}")
def get_report_section(report_id: str, section: str, client: ReportClient) -> str:
    # URI parameters automatically extracted and passed
    report = client.get_report(int(report_id))
    return report.get_section(section)
```

## 🐛 **Debugging**

### **Common Issues**

**Agent only sees one tool?**
```python
# Check: Are all decorators at module level?
# ✅ CORRECT
@framework.tool()
def my_tool(): pass

# ❌ WRONG - inside function
def setup():
    @framework.tool()  # Won't work!
    def hidden_tool(): pass
```

**Tools not discovered?**
```bash
# Test locally first
python my_mcp_server.py

# Should show all tools
# If not, check for syntax errors
```

**Client injection failing?**
```python
# Check: Type hints must be exact
def my_tool(client: MyAPIClient):  # ✅ Correct
def my_tool(client):               # ❌ No type hint
def my_tool(client: "MyAPIClient"): # ❌ String annotation
```

### **Debug Mode**
```python
# Enable debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

# Framework will log:
# 🔧 Registered tool: my_tool
# 🚀 Handling event: tools/list  
# 📊 Available tools: ['tool1', 'tool2']
```

## 📖 **API Reference**

### **Framework Class**
```python
framework = MCPLambdaFramework(name="My Server", version="1.0.0")
```

### **Decorators**
```python
@framework.tool(name="custom_name", description="Tool description")
@framework.resource(uri_template="myapi://item/{id}", description="Resource description")  
@framework.prompt(name="prompt_name", description="Prompt description")
```

### **Client Base Class**
```python
class MyClient(MCPClient):
    def authenticate(self) -> bool:
        # Return True if authenticated
        pass
    
    def _handle_error(self, error: Exception, context: str = "operation"):
        # Custom error handling
        pass
```

### **Lambda Handler**
```python
def lambda_handler(event, context):
    return framework.handle_lambda_event(event, context)
```

## 🤝 **Contributing**

We welcome contributions! Here's how to get started:

```bash
# Clone the repo
git clone https://github.com/your-org/Mcpylambda.git
cd Mcpylambda

# Run tests
python -m pytest tests/

# Run the example
python examples/calculator_server.py
```

### **Development Setup**
```bash
# Install dev dependencies
pip install -r requirements-dev.txt

# Run linting
black Mcpylambda/
flake8 Mcpylambda/

# Run type checking  
mypy Mcpylambda/
```

## 📄 **License**

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 **Acknowledgments**

- [Model Context Protocol](https://modelcontextprotocol.io/) - The protocol that makes this possible
- [AWS Lambda](https://aws.amazon.com/lambda/) - Serverless compute platform
- [Anthropic Claude](https://claude.ai/) - AI assistant that works great with MCP

## 🔗 **Links**

- **Documentation**: [https://Mcpylambda-mcp.readthedocs.io](https://Mcpylambda-mcp.readthedocs.io)
- **Examples**: [https://github.com/your-org/Mcpylambda/tree/main/examples](https://github.com/your-org/Mcpylambda/tree/main/examples)
- **Issues**: [https://github.com/your-org/Mcpylambda/issues](https://github.com/your-org/Mcpylambda/issues)
- **Discussions**: [https://github.com/your-org/Mcpylambda/discussions](https://github.com/your-org/Mcpylambda/discussions)

---

**Made with ❤️ for the AI agent ecosystem**

*Transform your APIs into AI superpowers with Mcpylambda* 🎼
