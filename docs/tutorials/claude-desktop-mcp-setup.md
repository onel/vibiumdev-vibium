# Setting Up Vibium MCP in Claude Desktop

This tutorial covers how to configure Vibium as an MCP (Model Context Protocol) server in Claude Desktop.

## Prerequisites

- Claude Desktop installed ([download](https://claude.ai/download))
- Node.js 18+ (for npx)

## Add Vibium MCP

### Option 1: Using npx (Recommended)

Edit your Claude Desktop configuration file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

**Linux:** `~/.config/Claude/claude_desktop_config.json`

Add Vibium to the `mcpServers` section:

```json
{
  "mcpServers": {
    "vibium": {
      "command": "npx",
      "args": ["-y", "vibium"]
    }
  }
}
```

If the file doesn't exist, create it with the content above.

### Option 2: Using Local Binary

If you built clicker locally:

```json
{
  "mcpServers": {
    "vibium": {
      "command": "/path/to/clicker",
      "args": ["mcp"]
    }
  }
}
```

For example, from the vibium repo root:

```json
{
  "mcpServers": {
    "vibium": {
      "command": "/Users/yourname/Projects/vibium/clicker/bin/clicker",
      "args": ["mcp"]
    }
  }
}
```

### Custom Screenshot Directory

By default, screenshots are saved to:
- macOS: `~/Pictures/Vibium/`
- Linux: `~/Pictures/Vibium/`
- Windows: `%USERPROFILE%\Pictures\Vibium\`

To use a different directory:

```json
{
  "mcpServers": {
    "vibium": {
      "command": "npx",
      "args": ["-y", "vibium", "--screenshot-dir", "./screenshots"]
    }
  }
}
```

To disable file saving (base64 inline only):

```json
{
  "mcpServers": {
    "vibium": {
      "command": "npx",
      "args": ["-y", "vibium", "--screenshot-dir", ""]
    }
  }
}
```

### Verify Installation

After editing the configuration file:

1. **Restart Claude Desktop** completely (quit and reopen)
2. Start a new conversation
3. Look for the MCP tools icon (🔌) in the input area
4. Click it to see available MCP servers — Vibium should be listed

## Testing the Integration

Once configured, ask Claude to use browser automation:

```
Take a screenshot of https://example.com
```

Claude will use the Vibium MCP tools:
1. `browser_launch` - Start the browser
2. `browser_navigate` - Go to the URL
3. `browser_screenshot` - Capture the page
4. `browser_quit` - Close the browser

## Available MCP Tools

| Tool | Description |
|------|-------------|
| `browser_launch` | Start a browser session (visible by default) |
| `browser_navigate` | Navigate to a URL |
| `browser_click` | Click an element by CSS selector |
| `browser_type` | Type text into an element |
| `browser_screenshot` | Capture a screenshot |
| `browser_find` | Find element info (tag, text, bounding box) |
| `browser_quit` | Close the browser session |

## Troubleshooting

### MCP server not appearing

1. **Check the configuration file path** — make sure you edited the correct file for your OS
2. **Verify JSON syntax** — use a JSON validator to check for syntax errors
3. **Restart Claude Desktop** — changes only take effect after a full restart
4. **Check Claude Desktop logs** for error messages:
   - macOS: `~/Library/Logs/Claude/`
   - Windows: `%APPDATA%\Claude\logs\`
   - Linux: `~/.config/Claude/logs/`

### "MCP server failed to start" error

This usually means the command isn't running correctly. Try:

1. **Verify npx works:**
   ```bash
   npx -y vibium
   ```
   You should see "Vibium MCP server" output (press Ctrl+C to exit).

2. **Use absolute paths** instead of relative paths in the configuration

3. **Check Node.js version:**
   ```bash
   node --version
   ```
   Should be 18 or higher.

### Browser fails to launch

The first time you run Vibium, it downloads Chrome for Testing. If this fails:

```bash
# Run directly to see error messages
npx -y vibium
```

On macOS, if you see a Gatekeeper warning about chromedriver, this should be fixed in v0.1.5+.

### Test MCP server manually

Send JSON-RPC messages directly to verify the server works:

```bash
cat << 'EOF' | npx -y vibium
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"capabilities":{}}}
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"browser_launch","arguments":{"headless":true}}}
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"browser_navigate","arguments":{"url":"https://example.com"}}}
{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{"name":"browser_screenshot","arguments":{}}}
{"jsonrpc":"2.0","id":5,"method":"tools/call","params":{"name":"browser_quit","arguments":{}}}
EOF
```

You should see JSON responses for each command.

### View detailed logs

Run the MCP server directly to see detailed output:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"capabilities":{}}}' | npx -y vibium
```

Expected response:
```json
{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2024-11-05","capabilities":{"tools":{}},"serverInfo":{"name":"vibium","version":"0.1.0"}}}
```

## Remove Vibium MCP

Edit your `claude_desktop_config.json` and remove the `vibium` entry from `mcpServers`, then restart Claude Desktop.

## References

- [Claude Desktop MCP Documentation](https://modelcontextprotocol.io/docs/tools/claude-desktop)
- [Model Context Protocol Specification](https://spec.modelcontextprotocol.io/)
