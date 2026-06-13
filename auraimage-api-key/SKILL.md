---
name: auraimage-api-key
description: Set up, fix, or rotate an existing AuraImage Secret Key (and project name) in a project's env store — any language or framework (.env, .env.local, .dev.vars, etc.). Use whenever the user wants to wire an AuraImage key into a project, fix a key that "isn't working" (e.g. a 401 invalid-token on upload), or rotate a leaked/old key — phrasings like "set up my auraimage api key", "add my auraimage secret key to this app", "configure AURA_SECRET_KEY", "my auraimage AURA_SECRET_KEY is stale", "rotate my auraimage key". Prefer this over hand-editing env files: it validates the key against the AuraImage API, masks it, refuses to write into git-tracked files, and fixes .gitignore. Scope guards — only AuraImage (not OpenAI/Stripe/other services' keys), only wires EXISTING keys (it never generates or mints new keys; those come from the dashboard), and it is neither the full installer (install-auraimage-app) nor the MCP setup (auraimage-mcp), though both delegate their secret-writing to it. The only skill in the collection allowed to write secret values to disk.
license: MIT
metadata:
  author: auraimage
  version: "0.1.0"
---

# AuraImage Secret Key Setup

Writes exactly two values into the project's gitignored env store: the **Secret Key** (`AURA_SECRET_KEY=sk_live_…`) and the **project name** (`AURA_PROJECT_NAME=…`). Works in any language or ecosystem — Node, Python, Ruby, Go, PHP, Rust — because it adapts to the project's existing env convention instead of assuming one.

Terminology: the credential is the **Secret Key** (that's its name in the dashboard and docs). "API key" appears only in this skill's name, as the colloquial alias people search for. Say "Secret Key" in everything you show the user.

## Contract

This skill is the collection's designated secret-handler. The contract exists so that *one* place owns the risky write and can be audited:

- **Sole writer.** No other AuraImage skill writes secret material to disk. When another skill (the installer, the MCP setup) needs the key wired, it invokes this skill rather than writing the file itself.
- **Never write unconfirmed.** Before any write, show the user the masked value, the exact target file, and what will change — and get an explicit OK. This holds even when invoked from another skill's already-confirmed flow; the caller's plan confirm does not substitute. The invariant must be unconditional or it isn't an invariant.
- **Find-or-prompt; never mint.** Keys are created in the dashboard (or by `aura init`), not here. If the user has no key, send them to **https://app.auraimage.ai → your project → Settings → Secret Keys** (no account: https://auraimage.ai/signup).
- **Validate before writing.** A credentials skill that writes a broken credential creates a failure the user discovers much later, far from the cause. One API call at setup time prevents that.
- **Idempotent.** Re-running against a correctly configured project changes nothing and says so.
- **Never echo the full key.** In confirms, reports, and errors, show `sk_live_abcd…1234` (first 4 / last 4 of the suffix). The plaintext appears nowhere except the env file itself.

## Step 1 — Discover the env store

Follow `rules/store-selection.md`. It determines:

- the **target file** (existing store wins; `.env` is the default for new setups; small framework override table),
- the **variable names** to write (canonical `AURA_SECRET_KEY` + `AURA_PROJECT_NAME`; an existing project's legacy names win; Next.js gets `NEXT_PUBLIC_AURA_PROJECT_NAME` for the project name),
- the **safety gates** (abort if the target file is tracked by git; gitignore coverage; files never to touch).

## Step 2 — Find or prompt the credentials

Scan the candidate stores for an existing Secret Key (canonical or legacy name):

- **Found:** confirm with a masked preview — *"Found existing `AURA_SECRET_KEY=sk_live_abcd…1234` in `.env`. Use it? [Y/n]"*. "n" means the user wants a different key — treat as rotation: prompt for the new value.
- **Not found:** prompt — *"Paste your AuraImage Secret Key (starts with `sk_live_`). Find or create one at https://app.auraimage.ai → your project → Settings → Secret Keys."*

Then the project name, if not already present: *"What is your AuraImage project name? It's the project's name in the dashboard."*

## Step 3 — Validate

Follow `rules/validation.md`: a local format check, then one `POST /v1/sign` call that verifies the key **and** that it belongs to the project name the user gave (the key itself identifies the project server-side, so a mismatch is caught here instead of at first upload). Offline or unreachable API degrades to a warning plus an explicit write-anyway choice.

## Step 4 — Confirm the write

One screen, then one Y/n:

```
Ready to write:

  AURA_SECRET_KEY     sk_live_abcd…1234   (validated ✓)
  AURA_PROJECT_NAME   my-project

  → .env  (gitignored ✓, not tracked by git ✓)

Proceed? [Y/n]
```

Adapt the variable names and target file to what discovery chose. If validation was skipped (offline), the first line says `(not validated — API unreachable)` instead.

## Step 5 — Write

Follow `rules/env-writes.md`: line-preserving idempotent upsert, gitignore amendment, and the rule that the Secret Key is **never** written under a client-exposure prefix (`NEXT_PUBLIC_`, `VITE_`, …) — a public prefix compiles the value into browser bundles.

## Step 6 — Report

Short and concrete:

- **Wrote** — delta lines (`.env +2 keys`, `.gitignore +1 line`), or "Already configured, nothing changed."
- **Production** — the env file covers local dev only. If a deploy target was detected, print (never run) the platform's secret command:
  | Detected | Command |
  |---|---|
  | `vercel.json` | `vercel env add AURA_SECRET_KEY` |
  | `wrangler.jsonc` / `wrangler.toml` | `npx wrangler secret put AURA_SECRET_KEY` |
  | `netlify.toml` | `netlify env:set AURA_SECRET_KEY <value>` |
  | `fly.toml` | `fly secrets set AURA_SECRET_KEY=<value>` |
  | `Procfile` (Heroku) | `heroku config:set AURA_SECRET_KEY=<value>` |
- **Loading** — if the detected stack doesn't auto-load `.env` files (plain Go, Java, shell scripts), say so with the one-liner that does (`godotenv`, `direnv allow`, or `set -a; . ./.env; set +a`). Stacks with built-in dotenv support (Next.js, Vite, Rails, Laravel) need no note.

## Failure handling

- **Validation says the key is bad (401):** re-prompt once or twice; if the user insists the key is right, offer write-anyway with a warning — the API may lag a just-created key.
- **Validation says wrong project (403):** relay the server's message — the key belongs to a different project than the name given. Ask which one is wrong.
- **Target file is tracked by git:** abort the write, explain that a gitignore entry can't protect an already-tracked file, and give the fix (`git rm --cached <file>`, commit, then re-run). Do not write a secret into a tracked file under any circumstances.
- **Any write fails midway:** stop, report exactly what was and wasn't written. Re-running resumes safely because every write is an upsert.

## Re-run and rotation

Re-run with everything already set and matching: print *"Already configured: `AURA_SECRET_KEY=sk_live_abcd…1234` in `.env`."* — offer to validate it, change nothing.

Rotation ("rotate my key", "replace my key", or answering "n" to the found-key confirm): prompt for the new value, validate, replace in place. Remind the user to revoke the old key in the dashboard — rotation isn't complete until the old key is dead.

## Composition

`install-auraimage-app` and `auraimage-mcp` invoke this skill instead of writing env files themselves. When invoked that way, run the full flow including the write confirm (see Contract), but skip discovery the caller already did if its results are available — don't re-scan for the sake of it. Return control to the caller after Step 6 so it can continue its own plan.
