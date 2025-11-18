# Claude Desktop Integration

1. Open Claude Desktop settings and add a new MCP server.
2. Command: `python`
3. Args: `-m mcp_hub.server`
4. Environment:
   - `DEVOPS_COPILOT_CONFIG=/path/to/config/mcp.config.json`
   - `LLM_CONFIG=/path/to/config/llm.config.json`
5. Restart Claude Desktop and enable the server.
6. Prompt Claude to use the DevOps tools (e.g., ask for a Terraform review).
