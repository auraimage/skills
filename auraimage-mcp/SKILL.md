---
name: auraimage-mcp
description: Sets up the AuraImage MCP server for the current project. Use when a user asks to "set up AuraImage MCP", "install the AuraImage server", "configure AuraImage tools", or when audit_lcp / migrate_assets / generate_responsive_tag tools are not available and the user wants to use them. Writes the MCP server config to .mcp.json automatically.
license: MIT
metadata:
  author: auraimage
  version: "0.1.0"
---

# AuraImage MCP Setup

Configure the `@auraimage/mcp-server` so your AI coding agent has direct access to AuraImage tools: `audit_lcp`, `migrate_assets`, `generate_alt`, `generate_responsive_tag`, and `smart_crop_preview`.

## What the MCP Server Provides

| Tool | Description |
|------|-------------|
| `audit_lcp` | Scans a project directory for unoptimized images and estimates LCP savings |
| `migrate_assets` | Uploads local images to AuraImage (supports dry-run preview) |
| `generate_alt` | Generates accessible alt text for an image URL using vision AI |
| `generate_responsive_tag` | Generates a `<picture>` element with JXL/AVIF srcsets |
| `smart_crop_preview` | Returns crop variant URLs for an image at specified dimensions |

## Setup Instructions (Agent: Follow These Steps)

### Step 1 — Check for an existing `.mcp.json`

Look for `.mcp.json` in the project root. If it exists, read it first so you can merge rather than overwrite.

### Step 2 — Add the AuraImage server entry

Add the following entry under `mcpServers` in `.mcp.json`:

```json
{
  "mcpServers": {
    "auraimage": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@auraimage/mcp-server@latest"],
      "env": {
        "AURA_SECRET_KEY": "${AURA_SECRET_KEY}"
      }
    }
  }
}
```

If `.mcp.json` does not exist, create it with exactly the content above.

If it already exists, merge the `"auraimage"` key into the existing `mcpServers` object without removing other entries.

### Step 3 — Ensure the secret key is available

The MCP server needs `AURA_SECRET_KEY` to sign upload tokens. Ask the user:

> "What is your AuraImage secret key? You can find it in the AuraImage dashboard under **Settings → API Keys**."

Once provided, add it to the project's `.env.local` (Next.js) or `.env` file:

```
AURA_SECRET_KEY=sk_live_...
```

Make sure `.env.local` and `.env` are in `.gitignore` — **never commit the secret key**.

### Step 4 — Confirm the setup

Tell the user: "The AuraImage MCP server is configured. Restart your AI coding agent (or reload the MCP server list) to activate the tools. You can verify by asking your agent to run `audit_lcp` on this project."

## Complete `.mcp.json` Example

If the project has no existing `.mcp.json`:

```json
{
  "mcpServers": {
    "auraimage": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@auraimage/mcp-server@latest"],
      "env": {
        "AURA_SECRET_KEY": "${AURA_SECRET_KEY}"
      }
    }
  }
}
```

## Merging With an Existing `.mcp.json`

If `.mcp.json` already contains other servers (e.g. `shadcn`, `OpenAPI`), preserve them:

```json
{
  "mcpServers": {
    "existing-server": { "...": "..." },
    "auraimage": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@auraimage/mcp-server@latest"],
      "env": {
        "AURA_SECRET_KEY": "${AURA_SECRET_KEY}"
      }
    }
  }
}
```

## Notes

- `npx -y` downloads and runs the latest version automatically — no global install needed.
- The server uses stdio transport, which is compatible with Claude Code, Cursor, Cline, and all other MCP-compatible agents.
- `AURA_SECRET_KEY` is only needed for `migrate_assets`. The other tools (`audit_lcp`, `generate_responsive_tag`, `smart_crop_preview`, `generate_alt`) work without it.
