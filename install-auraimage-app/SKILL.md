---
name: install-auraimage-app
description: Install and set up AuraImage in the user's project. Use when the user wants to install, set up, integrate, or add AuraImage to their app. Triggers on phrases like "install AuraImage", "set up AuraImage", "add AuraImage to this project", "/install-auraimage-app". The entry point for new AuraImage installations — orchestrates SDK install, MCP server registration, React component install, env var setup, and token route scaffolding behind a single confirm prompt.
license: MIT
metadata:
  author: auraimage
  version: "0.1.0"
---

# Install AuraImage

End-to-end installer for adding AuraImage to a customer's project. Runs five steps behind one confirmation: discover → credentials → env writes → MCP merge → SDK install → component install → token route → final report.

## Operating Principles

- **One confirm, not N.** Show a discovery summary + ordered plan, ask once, then execute.
- **Idempotent.** Every write is read-merge-write. Re-running the skill on a fully-installed project prints "Already installed" and exits clean.
- **Degrade and report.** When the project can't support a step (no backend → no uploader, unknown framework → no components), skip it with a reason. Never abort mid-flow for a per-step gap.
- **Stop and report on failure.** On the first failed write, halt. Leave successful writes in place; tell the user exactly which step failed and the command to retry. Re-running resumes from the failed step.
- **Never auto-run side effects after install.** No `audit_lcp`, no `migrate_assets`. Surface them in the final report and let the user decide.

## Pre-Flight (abort before any writes if any of these are true)

1. No `package.json` in cwd → abort: *"No `package.json` found. Run this skill from the project root."*
2. `package.json` present but no detectable framework, no backend, no images anywhere → abort: *"No installable surface detected (no framework, no backend, no image usage). Nothing to do."*
3. Detected package-manager binary not on PATH → abort with the tailored message:
   - pnpm/yarn missing → *"`<pm>` is declared in `package.json` but not installed. Run `corepack enable` then re-invoke /install-auraimage-app."*
   - npm missing → *"`npm` not found. Reinstall Node.js."*
   - bun missing → *"`bun` is declared but not installed. See https://bun.sh/install."*

Pre-flight failure is different from per-step degradation: the discovery summary never prints.

## Step 1 — Discover

Run the five-dimension scan documented in `rules/discovery.md`:

1. **Framework & runtime** — `package.json` deps, Next.js version + router (`app/` vs `pages/`), TypeScript, package manager.
2. **Image surface** — `<img>`, `<Image>`, `<picture>`, `background-image: url(...)`, files in `/public`, `/assets`, `/src/assets`, third-party CDN URLs (`res.cloudinary.com`, `imgix.net`, `*.s3.*`, `imagekit.io`).
3. **Secrets & config** — `.env`, `.env.local`, `.env.example`, `.gitignore` coverage, existing `AURA_*` keys, existing `.mcp.json`, existing `components.json`.
4. **Backend** — does the project have a runtime that can sign upload tokens? (Next.js API/route handlers, Express, Hono, Fastify, plain Node server). Pure SPA = no backend.
5. **Hosting target** — `vercel.json`, `wrangler.jsonc`, `netlify.toml`, `open-next.config.ts`. Drives env-file convention.

## Step 2 — Credentials

Find-or-prompt `AURA_SECRET_KEY` and `NEXT_PUBLIC_AURA_PROJECT_NAME`.

- If `AURA_SECRET_KEY` exists in `.env.local` / `.env`, confirm with masked preview: *"Found existing `AURA_SECRET_KEY=sk_live_abcd…1234`. Use it? [Y/n]"*
- Otherwise prompt: *"What is your AuraImage secret key? Find it at https://app.auraimage.ai → Settings → API Keys."*
- Then prompt for projectName: *"What is your AuraImage project projectName? Find it in the dashboard → your project → Settings."*

If the user has no account, point them at https://auraimage.ai/signup before continuing.

## Step 3 — Show plan, get one confirm

Render the discovery + plan screen documented in `rules/discovery.md`. Skipped steps appear as `~~strikethrough~~ (skipped: <reason>)`.

End with: *"Proceed? [Y/n] (or describe changes, e.g. 'skip uploader')"*

- `Y` / empty → execute.
- `n` → abort, no writes.
- Free-text edit → re-print plan with the change applied, ask again. Cap at one round of editing, then it's Y/n only.

## Step 4 — Execute (in this exact order)

Each step is independent. On failure of any step, stop immediately and print the stop-and-report message (see "Failure handling" below). Do not attempt later steps.

### 4a. Env writes

See `rules/env-writes.md`. Upsert `AURA_SECRET_KEY` and `NEXT_PUBLIC_AURA_PROJECT_NAME` into `.env.local` (Next.js / Vercel) or `.env` (everything else). Ensure that file is in `.gitignore`. Never touch `.env.example`.

### 4b. MCP server registration

Read existing `.mcp.json` (or create empty `{ "mcpServers": {} }` if missing). Merge — do not overwrite siblings:

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

If the `auraimage` key already exists with identical contents, no-op.

### 4c. SDK install

Run `<pm> add @auraimage/sdk` using the detected package manager (precedence: `packageManager` field → lockfile → most-recent-mtime tiebreaker → `npm`). Multiple lockfiles + no `packageManager` field → use the most recently modified lockfile and print a warning in the discovery summary: *"Multiple lockfiles found (X, Y). Using <pm> based on mtime. Delete the stale one to silence this warning."*

Skip if `@auraimage/sdk` is already in `dependencies`.

Deno (`deno.json` / `deno.lock`) is explicitly refused — skip with: *"Deno not supported by SDK installer. Install manually: `deno add npm:@auraimage/sdk` (untested)."*

### 4d. Component install

React or Next.js detected only. Run:

```sh
npx shadcn@latest add https://auraimage.ai/registry/image.json
```

If a backend was detected, also run:

```sh
npx shadcn@latest add https://auraimage.ai/registry/uploader.json
```

If no backend → skip the uploader with: *"no backend detected — install when you add API routes."*

### 4e. Token route scaffold

Next.js + uploader installed only. See `rules/token-route.md`. Create `app/api/aura/sign/route.ts` (app router) or `pages/api/aura/sign.ts` (pages router) using the template in that rule file. The path matches `<AuraUploader />`'s default sign endpoint, so the uploader works with no extra wiring.

If the file already exists, do **not** overwrite. Diff against the template and ask the user before touching.

For non-Next backends (Express, Hono, Fastify) detected from imports in `package.json`: print the framework-appropriate snippet from `rules/token-route.md` in the final report under "Next" — do not auto-write to those projects (the route's mount point is project-specific).

### 4f. Final report

Render per `rules/final-report.md`. Three sections:

- **Wrote** — every mutation as a delta line (`.env.local +2 keys`, `.mcp.json merged`, etc.).
- **Skipped** — every plan item that was dropped, with the reason.
- **Next** — at most 4 numbered actions: restart agent for MCP, verify with `audit_lcp`, migrate detected assets, the forward reference to `auraimage-react`.

End with the docs link: `Docs: https://auraimage.ai/docs`.

## Failure Handling

On any step failure during step 4:

```
✗ AuraImage install stopped at step <N> — <step name>.

Wrote (kept):
  <delta lines for steps that succeeded>

Failed:
  <step name> — <error message>
  Retry: <exact command the user can run>

Re-run /install-auraimage-app to resume from the failed step.
```

Do not roll back successful writes. Do not attempt subsequent steps.

## Composition With Sibling Skills

This skill inlines the install-time bits and **points** at sibling skills for ongoing use:

- `auraimage-react` — `<AuraImage />` props, triple-stage loading, uploader patterns. Activates automatically when editing image code post-install.
- `auraimage-url-api` — URL parameters, responsive `<picture>`, fit modes. Activates when editing raw `<img>` / `<picture>`.
- `auraimage-migrate` — `audit_lcp` / `migrate_assets` workflow.
- `auraimage-mcp` — the MCP server config (this skill performs the same merge inline; the sibling skill remains the canonical reference).

Reference these in the final report's "Next" section. Do not re-document their rules here.

## Re-Run Behavior

A second invocation against an already-installed project should:

1. Run discovery — finds `AURA_SECRET_KEY`, `auraimage` in `.mcp.json`, SDK in deps, components present, route exists.
2. Render plan with every step `~~strikethrough~~ (skipped: already installed)`.
3. Print: *"Already installed, nothing to do."*
4. Exit without prompting.
