# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a hybrid MCP (Model Context Protocol) server for Odoo ERP integration. It supports both:
- **HTTP Streaming**: For web-based AI agents and remote access
- **stdio**: For local AI assistants like Claude Desktop

## Architecture

### Dual Implementation

1. **HTTP Mode** (`http_server.py`):
   - FastAPI-based HTTP server
   - Supports ngrok for remote access
   - Includes caching and advanced features
   - Used for AI agents like Flowhunt

2. **stdio Mode** (`server.py`):
   - Standard MCP stdio implementation
   - Direct integration with Claude Desktop
   - Simpler, lightweight implementation

### Core Components

- **`odoo_service.py`**: Advanced service for HTTP mode with caching
- **`odoo_client.py`**: Simple client for stdio mode
- **`config.py`**: Configuration management with Pydantic
- **`cache_service.py`**: Caching system for HTTP mode

## Development Commands

```bash
# Install dependencies
pip install -e .
pip install -e ".[dev]"  # For development with test dependencies

# Run HTTP server
python -m mcp_server_odoo.http_server

# Run stdio server
python -m mcp_server_odoo

# Run tests
pytest
pytest --cov  # With coverage report

# Type checking
mypy mcp_server_odoo

# Linting
ruff check .
ruff format .
```

## Environment Configuration

### HTTP Mode
Uses comprehensive configuration via `config.py`:
- Odoo credentials
- Server settings
- Cache configuration

### stdio Mode
Uses simple environment variables:
- `ODOO_URL`
- `ODOO_DB`
- `ODOO_USERNAME`
- `ODOO_API_KEY` or `ODOO_PASSWORD`

## Usage Patterns

### HTTP Mode (AI Agents)
```python
# For Flowhunt and other AI platforms
# Configure MCP server URL: https://your-ngrok-url.ngrok-free.app
```

### stdio Mode (Claude Desktop)
```json
{
  "mcpServers": {
    "odoo": {
      "command": "python",
      "args": ["-m", "mcp_server_odoo"],
      "env": {
        "ODOO_URL": "https://your-instance.odoo.com",
        "ODOO_DB": "your-database",
        "ODOO_USERNAME": "your-email@example.com",
        "ODOO_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Testing Approach

- Unit tests for both implementations
- Integration tests with mock Odoo responses
- Test MCP protocol compliance for both modes
- Validate error handling and edge cases

## Security Considerations

- Never expose Odoo credentials in logs or error messages
- Validate and sanitize all inputs before sending to Odoo
- Use API keys instead of passwords when possible
- Implement proper SSL context handling
- Follow principle of least privilege for Odoo user permissions
