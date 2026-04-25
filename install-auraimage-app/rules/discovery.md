# Discovery — five-dimension scan

The discovery step runs before any user prompts. Output is the discovery summary screen rendered in step 3 of the SKILL.md flow.

## Dimension 1 — Framework & runtime

Read `package.json` once. Detect:

| Signal | Inference |
|--------|-----------|
| `dependencies.next` | Next.js — read version, then check `app/` vs `pages/` directory presence |
| `dependencies.react` (no `next`) | React (Vite / CRA / custom) — check `vite.config.*`, `craco.config.*` |
| `dependencies.vue` | Vue |
| `dependencies.svelte` or `@sveltejs/kit` | Svelte / SvelteKit |
| `dependencies.astro` | Astro |
| `dependencies.@remix-run/*` | Remix |
| None of the above | Plain Node / vanilla — degrade to URL-API-only mode |

TypeScript: presence of `tsconfig.json`.

Package manager precedence (this is the binary used in step 4c):

1. `package.json` `"packageManager"` field (e.g. `"pnpm@9.12.0"`).
2. Lockfile presence: `bun.lockb` → bun, `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `package-lock.json` → npm.
3. Multiple lockfiles + no `packageManager` field → most recently modified by mtime; warn in summary.
4. Fallback: `npm`.

## Dimension 2 — Image surface

Counts only — used to decide whether the final report's "Next" section recommends `migrate_assets`.

```sh
# Component image tags
rg -t tsx -t jsx -t ts -t js -t vue -t svelte -t astro -c '<img\b|<Image\b|<picture\b' src/ app/ pages/ components/ 2>/dev/null

# CSS background images
rg -t css -t scss -c 'background(-image)?:\s*url\(' src/ app/ styles/ 2>/dev/null

# Local image files
fd -e jpg -e jpeg -e png -e webp -e avif -e gif . public/ assets/ src/assets/ 2>/dev/null | wc -l

# Third-party CDN URLs
rg -c 'res\.cloudinary\.com|imgix\.net|imagekit\.io|\.s3\.[a-z0-9.-]+amazonaws\.com' src/ app/ pages/ components/ public/ 2>/dev/null
```

Report the totals; do not list every file.

## Dimension 3 — Secrets & config

Single read of each:

- `.env`, `.env.local`, `.env.development.local`, `.env.production.local` — does the file exist? Does it already contain `AURA_SECRET_KEY` or `NEXT_PUBLIC_AURA_SLUG`?
- `.env.example` — informational only; never written to.
- `.gitignore` — does it cover the env file we'll write to?
- `.mcp.json` — exists? If so, list the existing `mcpServers` keys for the summary.
- `components.json` — shadcn already configured?

## Dimension 4 — Backend surface

Decides whether `<AuraUploader />` and the token route can be installed.

| Signal | Backend? |
|--------|----------|
| Next.js `app/api/**/route.{ts,js}` exists | Yes — app router |
| Next.js `pages/api/**/*.{ts,js}` exists | Yes — pages router |
| Next.js detected, no API dirs | Yes — can scaffold (Next has a runtime) |
| `dependencies.express` / `hono` / `fastify` / `koa` | Yes — non-Next backend (no auto-scaffold; print snippet in report) |
| Pure Vite/CRA/Astro static / Svelte client-only | No — skip uploader, skip token route |

## Dimension 5 — Hosting target

Drives env-file convention and shows up in the summary.

| Signal | Target |
|--------|--------|
| `vercel.json` or no host file + Next.js | Vercel — write to `.env.local` |
| `wrangler.jsonc` / `wrangler.toml` or `open-next.config.ts` | Cloudflare Workers / Pages — write to `.env.local` for build, surface Wrangler-secret hint in report |
| `netlify.toml` | Netlify — write to `.env` |
| `Dockerfile` only | Self-hosted — write to `.env` |
| None | Default to `.env.local` for Next, `.env` otherwise |

## Discovery summary screen

```
AuraImage installer — discovered:

  Framework        Next.js 15 (app router) · TypeScript · pnpm
  Hosting          Cloudflare Workers (open-next.config.ts)
  Image surface    14 <Image> · 3 <picture> · 47 files in /public
                   3rd-party CDN: 8 res.cloudinary.com URLs
  Backend          Yes — app/api route handlers detected
  Existing config  .env.local present (no AURA_* keys)
                   .mcp.json present (1 server: shadcn)
                   .gitignore covers .env.local ✓

Plan:
  1. Add AURA_SECRET_KEY + NEXT_PUBLIC_AURA_SLUG to .env.local
  2. Merge "auraimage" into .mcp.json (keeps "shadcn")
  3. pnpm add @auraimage/sdk
  4. npx shadcn add aura/image + aura/uploader
  5. Create app/api/upload-token/route.ts
  6. Suggest: run migrate_assets on 8 Cloudinary URLs + /public

Proceed? [Y/n] (or describe changes, e.g. 'skip uploader')
```

Skipped plan items use `~~strikethrough~~ (skipped: <reason>)` instead of being omitted — so the user always sees the full intended scope and what was dropped.
