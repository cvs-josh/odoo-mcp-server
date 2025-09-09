# Odoo MCP Server (HTTP Streaming)

A Model Context Protocol (MCP) server that enables AI assistants to interact with Odoo ERP systems via HTTP streaming transport. This server provides tools for searching, creating, updating, and managing Odoo records through a standardized interface with full Docker containerization support.

## 🚀 Quick Start

### Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/vzeman/odoo-mcp-server-streamable-http.git
cd odoo-mcp-server-streamable-http

# Copy example and configure
cp docker-compose.example.yml docker-compose.yml
# Edit docker-compose.yml with your Odoo credentials

# Start with Docker Compose
docker-compose up -d

# Access the server
curl http://localhost:8000/health
```

### Local Development

```bash
# Install dependencies
pip install -e .

# Set environment variables
export ODOO_URL="https://your-instance.odoo.com"
export ODOO_DB="your-database"
export ODOO_USERNAME="your-email@example.com"
export ODOO_API_KEY="your-api-key"

# Run the server
python -m mcp_server_odoo.http_server
```

## Custom MCP Server Development
We develop MCP Servers for customers, if you need MCP server for your own system similar to Odoo MCP server, please contact us (https://www.flowhunt.io/contact/). 
Here is the description how we develop MCP Servers for our customers: https://www.flowhunt.io/services/mcp-server-development/

## Demo

📺 [Watch the demo on YouTube](https://youtu.be/tanzyt_qEmE)

## Features

- 🔍 **Search Records**: Query any Odoo model with complex domain filters
- ➕ **Create Records**: Add new records to any Odoo model
- ✏️ **Update Records**: Modify existing records
- 🗑️ **Delete Records**: Remove records from the system
- 📊 **Read Records**: Fetch detailed information about specific records
- 🔗 **Execute Methods**: Call custom methods on Odoo models
- 📋 **List Models**: Discover available models in your Odoo instance
- 🔧 **Model Introspection**: Get field definitions for any model
- 🌐 **HTTP Streaming**: Full MCP Streamable HTTP transport support
- 🐳 **Docker Ready**: Complete containerization with Docker Compose
- 🔄 **Server-Sent Events**: Real-time streaming responses
- 🛡️ **Security**: Origin validation, CORS support, and session management
- 📊 **Monitoring**: Health checks, metrics, and observability

## Installation

### 🐳 Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/vzeman/odoo-mcp-server-streamable-http.git
cd odoo-mcp-server-streamable-http

# Copy environment file and configure
cp .env.example .env
# Edit .env with your Odoo credentials

# Start with Docker Compose
docker-compose up -d
```

### 📦 Via pip

```bash
pip install odoo-mcp-server
```

### 🔧 From source

```bash
git clone https://github.com/vzeman/odoo-mcp-server-streamable-http.git
cd odoo-mcp-server-streamable-http
pip install -e .
```

## Configuration

### Environment Variables

Create a `.env` file in your project directory or set these environment variables:

```env
ODOO_URL=https://your-instance.odoo.com
ODOO_DB=your-database-name
ODOO_USERNAME=your-username@example.com
ODOO_API_KEY=your-api-key-here
```

### Getting Odoo Credentials

1. **API Key**: 
   - Log into your Odoo instance
   - Go to Settings → Users & Companies → Users
   - Select your user
   - Under "API Keys" or "Security" tab, create a new API key
   - Copy the key immediately (it won't be shown again)

2. **Database Name**:
   - Usually visible in the URL when logged in
   - Or check Settings → Activate Developer Mode → Database Info

3. **Username**:
   - Your login email address

## 🚀 Quick Start

### Docker Deployment

1. **Clone and configure**:
```bash
git clone https://github.com/vzeman/odoo-mcp-server-streamable-http.git
cd odoo-mcp-server-streamable-http
```

2. **Configure your Odoo credentials**:
```bash
# Copy the example file
cp docker-compose.example.yml docker-compose.yml

# Edit docker-compose.yml with your Odoo credentials
nano docker-compose.yml
```

Or edit the environment section directly:
```yaml
environment:
  - ODOO_URL=https://your-instance.odoo.com
  - ODOO_DB=your-database-name
  - ODOO_USERNAME=your-username@example.com
  - ODOO_API_KEY=your-api-key-here
```

3. **Start the services**:
```bash
# Basic deployment
docker-compose up -d

# With monitoring stack
docker-compose --profile monitoring up -d

# With nginx proxy
docker-compose --profile proxy up -d
```

4. **Access the server**:
- **MCP Endpoint**: `http://localhost:8000/mcp` or `http://localhost:8000/` (POST)
- **Health Check**: `http://localhost:8000/health`
- **API Documentation**: `http://localhost:8000/docs`
- **Server Info**: `http://localhost:8000/` (GET)

### Local Development

```bash
# Install dependencies
pip install -e .

# Set environment variables
export ODOO_URL="https://your-instance.odoo.com"
export ODOO_DB="your-database"
export ODOO_USERNAME="your-email@example.com"
export ODOO_API_KEY="your-api-key"

# Run the HTTP server
python -m mcp_server_odoo
```

## 🔌 Client Integration

### MCP Server Inspector

The server is fully compatible with the MCP Server Inspector. Use these connection settings:

- **URL**: `http://localhost:8000`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Endpoint**: `/` (root endpoint)

The server automatically handles both GET (server info) and POST (MCP protocol) requests on the root endpoint.

### HTTP Streaming Client

Connect to the MCP server using HTTP streaming transport:

```python
import httpx
import json

# Initialize session
response = httpx.post(
    "http://localhost:8000/mcp",
    json={
        "jsonrpc": "2.0",
        "id": 1,
        "method": "initialize",
        "params": {
            "protocolVersion": "2024-11-05",
            "capabilities": {},
            "clientInfo": {"name": "test-client", "version": "1.0.0"}
        }
    },
    headers={"Accept": "application/json, text/event-stream"}
)

# Get session ID from response headers
session_id = response.headers.get("Mcp-Session-Id")

# Make tool calls
response = httpx.post(
    "http://localhost:8000/mcp",
    json={
        "jsonrpc": "2.0",
        "id": 2,
        "method": "tools/call",
        "params": {
            "name": "search_records",
            "arguments": {
                "model": "res.partner",
                "domain": [["is_company", "=", True]],
                "limit": 10
            }
        }
    },
    headers={
        "Mcp-Session-Id": session_id,
        "Accept": "application/json, text/event-stream"
    }
```

### Server-Sent Events (SSE) Streaming

For real-time streaming responses:

```python
import httpx
import json

# Stream responses using SSE
with httpx.stream(
    "POST",
    "http://localhost:8000/mcp",
    json={
        "jsonrpc": "2.0",
        "id": 1,
        "method": "tools/call",
        "params": {
            "name": "search_records",
            "arguments": {
                "model": "res.partner",
                "domain": [["is_company", "=", True]],
                "limit": 10
            }
        }
    },
    headers={"Accept": "text/event-stream"}
) as response:
    for line in response.iter_lines():
        if line.startswith("data: "):
            data = json.loads(line[6:])  # Remove "data: " prefix
            print(f"Received: {data}")
```

### JSON Integration

The server provides a comprehensive JSON API for direct integration with any application. All MCP protocol methods are available via standard HTTP POST requests.

#### Basic JSON Request Format

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "method_name",
  "params": {
    "parameter1": "value1",
    "parameter2": "value2"
  }
}
```

#### Example: Initialize Connection

```bash
curl -X POST http://localhost:8000/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {"name": "my-app", "version": "1.0.0"}
    }
  }'
```

#### Example: List Available Tools

```bash
curl -X POST http://localhost:8000/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/list",
    "params": {}
  }'
```

#### Example: Call a Tool

```bash
curl -X POST http://localhost:8000/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "search_records",
      "arguments": {
        "model": "res.partner",
        "limit": 5
      }
    }
  }'
```

#### JavaScript/Node.js Integration

```javascript
const axios = require('axios');

class OdooMCPClient {
  constructor(baseUrl = 'http://localhost:8000') {
    this.baseUrl = baseUrl;
    this.requestId = 1;
  }

  async call(method, params = {}) {
    const response = await axios.post(this.baseUrl, {
      jsonrpc: "2.0",
      id: this.requestId++,
      method: method,
      params: params
    }, {
      headers: { 'Content-Type': 'application/json' }
    });
    
    return response.data.result;
  }

  async searchRecords(model, options = {}) {
    return this.call('tools/call', {
      name: 'search_records',
      arguments: { model, ...options }
    });
  }

  async createRecord(model, values) {
    return this.call('tools/call', {
      name: 'create_record',
      arguments: { model, values }
    });
  }
}

// Usage
const client = new OdooMCPClient();
const partners = await client.searchRecords('res.partner', { limit: 10 });
console.log(partners);
```

#### Python Integration

```python
import requests
import json

class OdooMCPClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
        self.request_id = 1
    
    def call(self, method, params=None):
        payload = {
            "jsonrpc": "2.0",
            "id": self.request_id,
            "method": method,
            "params": params or {}
        }
        self.request_id += 1
        
        response = requests.post(
            self.base_url,
            json=payload,
            headers={"Content-Type": "application/json"}
        )
        return response.json()["result"]
    
    def search_records(self, model, **options):
        return self.call("tools/call", {
            "name": "search_records",
            "arguments": {"model": model, **options}
        })
    
    def create_record(self, model, values):
        return self.call("tools/call", {
            "name": "create_record",
            "arguments": {"model": model, "values": values}
        })

# Usage
client = OdooMCPClient()
partners = client.search_records("res.partner", limit=10)
print(partners)
```

#### MCP Client Integration

For Claude Desktop, Cursor, or other MCP clients, use this configuration:

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "odoo-mcp-server": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://localhost:8000"
      ]
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "odoo-mcp-server": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://localhost:8000"
      ]
    }
  }
}
```

**Setup Steps:**
1. Start your MCP server: `docker-compose up -d`
2. Install mcp-remote: `npm install -g mcp-remote`
3. Add the configuration above to your MCP client
4. Restart your MCP client
5. You'll have access to all 12 Odoo tools!

**✅ Current Status:** 
- HTTP JSON-RPC endpoints work correctly via `curl`
- All 12 Odoo tools are fully functional
- Server connects to Odoo and executes operations successfully

**Working Configuration for JSON-RPC:**
```bash
# Test via curl (this works):
curl -X POST http://localhost:8000/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "search_count",
      "arguments": {"model": "res.partner"}
    }
  }'
```

## Available Tools

### search_records
Search for records in any Odoo model.

**Parameters:**
- `model` (required): The Odoo model name (e.g., 'res.partner', 'sale.order')
- `domain`: Odoo domain filter (default: [])
- `fields`: List of fields to return
- `limit`: Maximum number of records
- `offset`: Number of records to skip
- `order`: Sort order (e.g., 'name asc, id desc')

**Example prompts:**
- "Find all customers in California"
- "Show me sales orders from last month"
- "List products with stock quantity below 10"

### get_record
Get detailed information about specific records.

**Parameters:**
- `model` (required): The Odoo model name
- `ids` (required): List of record IDs
- `fields`: List of fields to return (optional)

**Example prompts:**
- "Show me details of customer with ID 42"
- "Get information about sales order SO0123"

### create_record
Create new records in Odoo.

**Parameters:**
- `model` (required): The Odoo model name
- `values` (required): Dictionary of field values

**Example prompts:**
- "Create a new customer named 'Acme Corp' with email contact@acme.com"
- "Add a new product called 'Widget Pro' with price $99.99"

### update_record
Update existing records.

**Parameters:**
- `model` (required): The Odoo model name
- `ids` (required): List of record IDs to update
- `values` (required): Dictionary of field values to update

**Example prompts:**
- "Update customer 42's phone number to +1-555-0123"
- "Change the status of sales order SO0123 to confirmed"

### delete_record
Delete records from Odoo (use with caution).

**Parameters:**
- `model` (required): The Odoo model name
- `ids` (required): List of record IDs to delete

**Example prompts:**
- "Delete the test customer record with ID 999"
- "Remove cancelled sales orders older than 2 years"

### execute_method
Execute custom methods on Odoo models.

**Parameters:**
- `model` (required): The Odoo model name
- `method` (required): Method name to execute
- `ids`: List of record IDs (if method requires)
- `args`: Additional positional arguments
- `kwargs`: Additional keyword arguments

**Example prompts:**
- "Confirm sales order SO0123"
- "Send invoice INV/2024/0001 by email"

### list_models
Discover available models in your Odoo instance.

**Parameters:**
- `transient`: Include transient (wizard) models (default: false)

**Example prompts:**
- "What models are available in Odoo?"
- "Show me all installed models"

### get_model_fields
Get field definitions for a model.

**Parameters:**
- `model` (required): The Odoo model name
- `fields`: Specific fields to get info for (optional)

**Example prompts:**
- "What fields are available on the customer model?"
- "Show me the structure of sales orders"

## Common Use Cases

### Customer Management
```
"Find all customers who haven't made a purchase in 6 months"
"Create a new contact for Acme Corp named John Doe"
"Update the credit limit for customer 'BigCo Industries'"
```

### Sales Operations
```
"Show me all draft quotations"
"Create a sales order for customer Acme Corp with 10 units of Product A"
"Convert quotation SO0123 to a confirmed order"
```

### Inventory Management
```
"List all products with stock below reorder point"
"Update the quantity on hand for SKU-12345"
"Find all pending stock moves"
```

### Financial Operations
```
"Show me all unpaid invoices"
"Create a vendor bill for Office Supplies Inc"
"List payments received today"
```

## Model Reference

Common Odoo models you can interact with:

- `res.partner` - Customers, suppliers, and contacts
- `sale.order` - Sales orders and quotations
- `account.move` - Invoices and bills
- `product.product` - Product variants
- `product.template` - Product templates
- `stock.move` - Stock movements
- `purchase.order` - Purchase orders
- `project.project` - Projects
- `project.task` - Project tasks
- `hr.employee` - Employees
- `res.users` - System users
- `res.company` - Companies

## 🐳 Docker Services

### Core Services

- **mcp-server**: Main MCP server with HTTP streaming
- **redis**: Session management and caching (optional)
- **nginx**: Reverse proxy with SSL termination (optional)

### Service Management

```bash
# Start basic services
docker-compose up -d

# Start with Redis for session management
docker-compose --profile redis up -d

# Start with Nginx reverse proxy
docker-compose --profile proxy up -d

# View logs
docker-compose logs -f mcp-server

# Scale services
docker-compose up -d --scale mcp-server=3

# Update services
docker-compose pull
docker-compose up -d

# Stop services
docker-compose down

# Clean up
docker-compose down -v --remove-orphans
```

### Endpoints

- **MCP Endpoint**: `http://localhost:8000/mcp` or `http://localhost:8000/` (POST)
- **Health Check**: `http://localhost:8000/health`
- **API Documentation**: `http://localhost:8000/docs`
- **Server Info**: `http://localhost:8000/` (GET)

## **Security Considerations**

### Production Deployment

- **Origin Validation**: Server validates Origin headers to prevent DNS rebinding
- **Localhost Binding**: Binds to 127.0.0.1 by default for local development
- **Session Management**: Secure session IDs with proper expiration
- **CORS Configuration**: Configurable CORS policies
- **Rate Limiting**: Nginx-based rate limiting
- **SSL/TLS**: HTTPS support via nginx reverse proxy

### Credentials Management

- Store credentials securely using environment variables
- Use API keys instead of passwords when possible
- Grant minimum necessary permissions to the API user
- Regularly rotate API keys
- Monitor API usage through Odoo's logs
- Use Docker secrets for sensitive data in production

## Troubleshooting

### Authentication Failed
- Verify your API key is active in Odoo
- Check if the username is correct (usually your email)
- Ensure the database name matches exactly

### Connection Refused
- Verify the Odoo URL (should not include `/web`)
- Check if your IP is whitelisted (if applicable)
- Ensure XML-RPC is enabled on your Odoo instance

### Model Not Found
- The model might require additional modules to be installed
- Use `list_models` to see available models
- Check if you have permissions for that model

## 🔧 Development

### Local Development Setup

```bash
# Clone repository
git clone https://github.com/vzeman/odoo-mcp-server-streamable-http.git
cd odoo-mcp-server-streamable-http

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in development mode
pip install -e ".[dev]"

# Set up environment
cp .env.example .env
# Edit .env with your configuration

# Run the server
python -m mcp_server_odoo
```

### Testing

```bash
# Install test dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Run with coverage
pytest --cov=mcp_server_odoo

# Test HTTP endpoints
curl -X POST http://localhost:8000/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

### Code Quality

```bash
# Type checking
mypy mcp_server_odoo

# Linting and formatting
ruff check .
ruff format .

# Security scanning
bandit -r mcp_server_odoo
```

### Docker Development

```bash
# Build development image
docker build -t odoo-mcp-server:dev .

# Run development container
docker run -p 8000:8000 \
  -e ODOO_URL="https://demo.odoo.com" \
  -e ODOO_DB="demo" \
  -e ODOO_USERNAME="demo" \
  -e ODOO_API_KEY="your-key" \
  odoo-mcp-server:dev

# Development with volume mounting
docker run -p 8000:8000 \
  -v $(pwd)/mcp_server_odoo:/app/mcp_server_odoo \
  -e ODOO_URL="https://demo.odoo.com" \
  odoo-mcp-server:dev
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and feature requests, please use the [GitHub issue tracker](https://github.com/vzeman/odoo-mcp-server/issues).