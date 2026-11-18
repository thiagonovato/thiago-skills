# MCP

An MCP server exposes **tools** (callable actions) and **resources** (readable data) to the agent over a **transport** — typically **stdio** (JSON-RPC on stdin/stdout).

Copy `mcp.json.example` to `mcp.json`, replace every `<PASTE_HERE>`, and keep secrets out of git (`mcp.json` is gitignored).
