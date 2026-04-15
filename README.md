# AuraImage Agent Skills

Skills for AI coding agents (Claude Code, Cursor, Copilot, Gemini CLI, Codex, Cline) that extend their ability to work with AuraImage.

## Install

```sh
npx skills add auraimage/skills
```

Or install individual skills:

```sh
npx skills add auraimage/skills/auraimage-url-api
npx skills add auraimage/skills/auraimage-react
npx skills add auraimage/skills/auraimage-migrate
npx skills add auraimage/skills/auraimage-mcp
```

---

## Available Skills

### `auraimage-react` ← Start here for React/Next.js

Rules for using the AuraImage Shadcn component registry.

**Use when:**
- Working in a React or Next.js project
- Displaying images with BlurHash placeholders
- Building an upload UI

**Covers:**
- `<AuraImage />` component with Triple-Stage Loading (BlurHash → LQIP → JXL)
- `<AuraUploader />` drag-and-drop widget
- BlurHash storage and integration pattern
- `placeholder` prop configuration per use case

---

### `auraimage-url-api`

Rules for constructing AuraImage transformation URLs and responsive `<picture>` elements.

**Use when:**
- Writing or reviewing any `<img>`, `<picture>`, or CSS `url()` that serves images
- Implementing responsive image layouts
- Choosing image formats or quality settings

**Covers:**
- URL format and all query parameters (`w`, `h`, `q`, `fmt`, `fit`)
- Responsive `<picture>` with JXL → AVIF srcsets
- Format auto-negotiation vs. explicit override
- Fit modes and tier requirements (`face` = Pro+, `auto` = Startup)

---

### `auraimage-migrate`

Step-by-step workflow for migrating local or third-party images to AuraImage.

**Use when:**
- A user asks to "migrate" or "optimize" images
- You detect local images in `/public`, `/assets`, or `require('./img')` patterns
- You see Cloudinary, Imgix, or S3 URLs that could be replaced

**Covers:**
- Audit → dry-run → upload workflow using MCP tools
- Detection of local and third-party image URLs
- JSX and CSS rewriting patterns
- Server-side HMAC signing for runtime uploads

---

### `auraimage-mcp`

Configures the `@auraimage/mcp-server` for the current project.

**Use when:**
- A user wants to use `audit_lcp`, `migrate_assets`, `generate_responsive_tag`, or other AuraImage tools
- Setting up AuraImage in a new project

**Covers:**
- Writing the correct entry to `.mcp.json`
- Merging with existing MCP server configs
- Setting up `AURA_SECRET_KEY` in `.env`

**MCP tools unlocked:**

| Tool | Description |
|------|-------------|
| `audit_lcp` | Scan for unoptimized images and estimate LCP savings |
| `migrate_assets` | Upload local images to AuraImage |
| `generate_alt` | Generate alt text via vision AI |
| `generate_responsive_tag` | Generate `<picture>` with JXL/AVIF srcsets |
| `smart_crop_preview` | Preview crop variants at specified dimensions |
