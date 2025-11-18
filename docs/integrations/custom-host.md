# Custom Host Integration

- Follow the MCP spec to build a JSON-RPC 2.0 client over stdio.
- Launch `python -m mcp_hub.server` as a subprocess.
- Negotiate capabilities and list available tools via the MCP protocol.
- Route user prompts or tool invocations through your host UI/CLI.
