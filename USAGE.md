# MCPyLambda Usage Guide

## Deploy Lambda from Git Repository

### 1. Clone Repository
```bash
git clone https://github.com/PushkarPahare/mcpylambda.git
cd mcpylambda
```

### 2. Create Lambda Function
```bash
# Create deployment package
zip -r mcpylambda.zip mcp_lambda_framework_single.py

# Create Lambda function
aws lambda create-function \
  --function-name mcpylambda-server \
  --runtime python3.9 \
  --role arn:aws:iam::YOUR-ACCOUNT:role/lambda-execution-role \
  --handler mcp_lambda_framework_single.lambda_handler \
  --zip-file fileb://mcpylambda.zip \
  --description "MCP server for AI agents. POST JSON-RPC 2.0 requests. Use {\"jsonrpc\":\"2.0\",\"method\":\"tools/list\",\"id\":1} to discover available tools, resources, and prompts."
```

### 3. Create API Gateway
```bash
# Create REST API
aws apigateway create-rest-api --name mcpylambda

# Get API ID and create resource/method
API_ID=$(aws apigateway get-rest-apis --query 'items[?name==`mcpylambda`].id' --output text)
RESOURCE_ID=$(aws apigateway get-resources --rest-api-id $API_ID --query 'items[0].id' --output text)

# Create POST method
aws apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --authorization-type NONE

# Integrate with Lambda
aws apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:YOUR-ACCOUNT:function:mcpylambda-server/invocations

# Deploy API
aws apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name prod
```

### 4. Test Deployment
```bash
# Test tools/list endpoint
curl -X POST https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/prod \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/list",
    "id": 1
  }'
```

## MCPyLambda Server Description

MCPyLambda is a Model Context Protocol (MCP) server that provides mathematical calculation tools and resources for AI agents. The server exposes a REST API endpoint that accepts MCP JSON-RPC requests.

### Entry API
- **Endpoint**: POST /mcp (or root path depending on API Gateway configuration)
- **Protocol**: MCP JSON-RPC 2.0a
- **Content-Type**: application/json

### Supported MCP Methods
- `initialize` - Initialize MCP session
- `tools/list` - List available tools
- `tools/call` - Execute a specific tool
- `resources/list` - List available resources  
- `resources/read` - Read a specific resource
- `prompts/list` - List available prompts
- `prompts/get` - Get a specific prompt

### Available Tools

The server exposes the following tools via the `tools/list` API:

- **hello_world**: Simple greeting function that accepts a name parameter
- **add**: Performs addition of two floating-point numbers
- **subtract**: Performs subtraction of two floating-point numbers  
- **multiply**: Performs multiplication of two floating-point numbers
- **divide**: Performs division with error handling for division by zero

### Tool Discovery

Agents can discover available tools by calling the `tools/list` method:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}
```

Response includes tool schemas with parameter types and requirements:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "add",
        "description": "Add two numbers",
        "inputSchema": {
          "type": "object",
          "properties": {
            "a": {"type": "number"},
            "b": {"type": "number"}
          },
          "required": ["a", "b"]
        }
      }
    ]
  }
}
```

### Resources

The server provides calculator history and operation details via resources:

- `calculator://history` - Complete calculation history
- `calculator://operation/{id}` - Specific operation details

### Client Configuration

Configure your MCP client to connect to the deployed Lambda:

```json
{
  "mcpServers": {
    "conductor": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-stdio", "https://YOUR-API-ID.execute-api.us-east-1.amazonaws.com/prod"]
    }
  }
}
```

## Environment Variables

Set these environment variables in Lambda configuration:

- `API_KEY` - Your API authentication key (if required)
- `LOG_LEVEL` - Logging level (DEBUG, INFO, WARNING, ERROR)

## Monitoring

Monitor Lambda function through CloudWatch:

```bash
# View logs
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/conductor-mcp-server

# View metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=conductor-mcp-server \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Sum
