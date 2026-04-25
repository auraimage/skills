# Final report

Step 4f of the SKILL.md flow. The last thing the skill prints. Three sections, in this exact order: **Wrote**, **Skipped**, **Next**.

## Format

```
✓ AuraImage installed.

Wrote:
  .env.local            +2 keys (AURA_SECRET_KEY, NEXT_PUBLIC_AURA_SLUG)
  .gitignore            +1 line (.env.local)
  .mcp.json             merged "auraimage" into mcpServers
  package.json          +1 dep (@auraimage/sdk@^<version>)
  components/aura/      image.tsx, uploader.tsx (via shadcn)
  app/api/upload-token/ route.ts (new)

Skipped:
  ~~Uploader component~~  (no backend detected — install when you add API routes)

Next:
  1. Restart your AI agent so the MCP server loads.
  2. Verify by asking: "run audit_lcp on this project"
  3. Migrate <N> Cloudinary URLs detected in src/: ask "run migrate_assets"
  4. Replace <img> / next/image with <AuraImage /> — the auraimage-react
     skill will activate automatically when you edit those files.

Docs: https://auraimage.ai/docs
```

## Rules for each section

### Wrote

- One line per mutation, deltas only — never state. `+2 keys` not `2 keys present`.
- File path on the left, change description on the right.
- Order: env files → config files (`.gitignore`, `.mcp.json`) → `package.json` → component files → scaffolded routes.
- If a step performed a no-op (e.g. SDK already installed, MCP entry already identical), do **not** list it under `Wrote`. List it under `Skipped` with reason `already present`.

### Skipped

- One line per skipped plan item, with reason in parentheses.
- Use `~~strikethrough~~` markdown so it visually mirrors the discovery plan.
- Reasons must be specific and actionable: *"no backend detected — install when you add API routes"*, *"already present"*, *"Deno runtime not supported"*.
- Omit the section entirely if nothing was skipped.

### Next

- Maximum 4 numbered actions.
- Order by what unlocks immediate value:
  1. Always: restart the agent (MCP server pickup).
  2. Always: verify with `audit_lcp`.
  3. Conditional on discovery dimension 2 finding 3rd-party CDN URLs or local images: migration suggestion with the count.
  4. Always: forward reference to the `auraimage-react` skill (or `auraimage-url-api` if React/Next was not detected).
- Cloudflare hosting target: append a 5th line if needed: *"For production: `pnpx wrangler secret put AURA_SECRET_KEY` from the relevant app directory."*
- End with `Docs: https://auraimage.ai/docs`.

## Re-run report (everything skipped)

```
✓ AuraImage already installed.

Skipped:
  ~~Add AURA_SECRET_KEY + NEXT_PUBLIC_AURA_SLUG~~ (skipped: already set)
  ~~Merge "auraimage" into .mcp.json~~ (skipped: already present)
  ~~Install @auraimage/sdk~~ (skipped: already in dependencies)
  ~~Install image + uploader components~~ (skipped: already present)
  ~~Create app/api/upload-token/route.ts~~ (skipped: file exists)

Nothing to do.
```

No `Next` section on a fully-installed re-run — the user already received it the first time.
