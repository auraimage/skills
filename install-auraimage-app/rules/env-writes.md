# Env writes — idempotent upsert

Step 4a of the SKILL.md flow. Writes `AURA_SECRET_KEY` and `NEXT_PUBLIC_AURA_PROJECT_SLUG` into the project's env file.

## Target file selection

Determined by the hosting target from discovery dimension 5:

- Vercel / Cloudflare / Next.js (default) → `.env.local`
- Netlify / self-hosted Docker / non-Next → `.env`
- Never write to `.env.example` (that file is for the project's contributors).
- Never write to `.env.development.local` / `.env.production.local` (too narrow).

If the target file does not exist, create it.

## Idempotent upsert algorithm

Read the file, parse line-by-line, replace any existing line whose key matches one of `AURA_SECRET_KEY` / `NEXT_PUBLIC_AURA_PROJECT_SLUG`, append the rest. Never blind-append (creates duplicates on re-run).

```
For each key in [AURA_SECRET_KEY, NEXT_PUBLIC_AURA_PROJECT_SLUG]:
  if a line in the file starts with `<key>=`:
    if existing value == new value → no-op for this key
    else → replace the line in place
  else:
    append `<key>=<value>` at end of file (with leading newline if file doesn't end in one)
```

Preserve all other lines, comments, and blank lines exactly.

## .gitignore amendment

Read `.gitignore`. If the chosen env file (e.g. `.env.local`) is not already covered by an existing line (literal match or glob like `.env*`), append it:

```
# AuraImage
.env.local
```

Skip if `.gitignore` does not exist (don't create one — that's a project-policy decision the installer shouldn't make on its own).

## Cloudflare Workers note

When hosting target is Cloudflare (`wrangler.jsonc` / `open-next.config.ts` detected), `.env.local` covers the local Next dev server and the build, but production secrets must be set via Wrangler. Do not run `wrangler secret put` from the installer — surface this in the final report instead:

> For production: set the secret in Cloudflare too: `pnpx wrangler secret put AURA_SECRET_KEY` from the relevant app directory.

## Re-run behavior

On re-run with identical values present, the upsert is a no-op for both keys. The discovery summary should detect "AURA_* keys present and match" and mark step 1 of the plan as `~~Add AURA_SECRET_KEY + NEXT_PUBLIC_AURA_PROJECT_SLUG~~ (skipped: already set)`.
