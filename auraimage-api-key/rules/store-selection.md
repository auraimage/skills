# Store selection — target file, variable names, safety gates

Step 1 of the SKILL.md flow. Everything here is read-only; the write happens in `rules/env-writes.md` after the user confirms.

## Target file

**An existing convention always wins.** A project with a working env store should never end up with a second one because a tool preferred a different filename. Check in this order and use the first that exists:

1. `.env.local`
2. `.env`
3. `.dev.vars` (Cloudflare Workers local dev)
4. `.envrc` (direnv — note: entries need `export ` prefixes; match the file's existing style)

If none exists, create one by framework:

| Detected | New file |
|---|---|
| Next.js (`next` in `package.json`) | `.env.local` |
| Wrangler project without a web framework (`wrangler.jsonc`/`wrangler.toml`, no `next`/`vite`/etc.) | `.dev.vars` |
| Everything else — any language | `.env` |

`.env` is the lingua franca: python-dotenv, dotenv (Ruby/Node), godotenv, phpdotenv, and most deploy tooling all read it. Don't invent per-language conventions beyond this table.

## Variable names

Canonical names, every language: `AURA_SECRET_KEY` and `AURA_PROJECT_NAME`.

Two exceptions:

- **Existing names win.** If the project already uses a legacy name (`AURAIMAGE_SECRET_KEY`, `AURAIMAGE_PROJECT_NAME`, `AURA_PROJECT`) *and code in the project reads it*, upsert the value under the existing name — renaming the variable breaks the code that reads it, and that's not this skill's call. Mention the legacy naming in the report; offer to migrate (rename var + update the code references) only if the user asks.
- **Client-exposure prefixes — project name only.** Frameworks that expose prefixed vars to browser code get the project name under their public variant, because client-side URL building needs it:

  | Framework | Project-name variable |
  |---|---|
  | Next.js | `NEXT_PUBLIC_AURA_PROJECT_NAME` |
  | Vite | `VITE_AURA_PROJECT_NAME` |
  | Nuxt | `NUXT_PUBLIC_AURA_PROJECT_NAME` |
  | Create React App | `REACT_APP_AURA_PROJECT_NAME` |
  | SvelteKit | `PUBLIC_AURA_PROJECT_NAME` |
  | Expo | `EXPO_PUBLIC_AURA_PROJECT_NAME` |

  Write the public variant *instead of* the plain name (these frameworks read prefixed vars server-side too — two copies of the same value drift).

  **The Secret Key never takes a public prefix.** A prefixed var is compiled into the client bundle; a prefixed secret is a published secret. If the user explicitly asks for `NEXT_PUBLIC_AURA_SECRET_KEY` (or any prefixed secret), refuse and explain why.

## Safety gates

Run all three before proposing the write:

1. **Git-tracked check.** `git ls-files --error-unmatch <target-file>` (exit 0 = tracked). A tracked file is an immediate abort for the secret — gitignore entries don't untrack files, and the secret would land in history. Tell the user the fix: `git rm --cached <file>`, commit, re-run. Not a git repo at all → skip this check.
2. **Gitignore coverage.** If `.gitignore` exists and no existing line covers the target file (literal or glob like `.env*`), plan to append it (the actual append happens in env-writes). If `.gitignore` doesn't exist, don't create one — whether to use git is the project's decision, not this skill's. Warn instead.
3. **Files never to touch.** `.env.example`, `.env.sample`, `.env.template` (they're documentation for contributors — committed by design), `.env.production*` / `.env.development*` (too narrow; surprise behavior across environments), and shell profiles (`.bashrc`, `.zshrc`, …) — a project credential doesn't belong in user-global state.

## Deploy-target detection

Purely informational — feeds the report's "Production" section in SKILL.md Step 6. Look for `vercel.json`, `wrangler.jsonc`/`wrangler.toml`, `netlify.toml`, `fly.toml`, `Procfile`. Never run a deploy platform's CLI from this skill; print the command for the user.
