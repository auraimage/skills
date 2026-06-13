# Validation — catch broken credentials at setup time, not first upload

Step 3 of the SKILL.md flow. The worst thing a credentials skill can do is silently write a typo'd, revoked, or wrong-project key — the user hits a confusing 401 days later, far from the cause. Two checks prevent that.

## 1. Local format check (always)

- Must start with `sk_live_`. Anything else — `aura_pat_…` (that's a CLI/MCP token), `pk_…` (public key), `psk_…` — is a different credential; tell the user which one they pasted and where the Secret Key lives (dashboard → project → Settings → Secret Keys).
- Strip surrounding whitespace and quotes from the paste before using it.

## 2. Live pair validation (when the network allows)

One request validates both the key and its binding to the project name the user gave — the key itself identifies the project server-side:

```sh
curl -s -o /dev/null -w '%{http_code}' -X POST https://api.auraimage.ai/v1/sign \
  -H "Authorization: Bearer $AURA_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{"projectName":"<the name the user gave>"}'
```

Pass the key via an environment variable or stdin — don't inline the plaintext into the command string (it lands in shell history and process lists).

| Status | Meaning | Action |
|---|---|---|
| 200 | Key valid, project matches | Proceed; mark `(validated ✓)` on the confirm screen |
| 401 | Key invalid or revoked | Re-prompt (max 2 retries). If the user insists it's correct, offer write-anyway with a warning — a just-created key may lag |
| 403 | Key valid but belongs to a **different project** | Relay the server message; ask whether the key or the project name is wrong; re-prompt for the wrong half |
| 402 | Key valid; account is hard-stopped (quota/overage limit reached) | Warn, then **proceed** — the credential is correct; the account state is a billing matter, not a setup error |
| Anything else / timeout / no network | Can't validate | Say so, offer the choice: write anyway `(not validated)` or stop and retry later |

The call's side effects are negligible: it mints one short-lived upload token (never used) and bumps the key's `lastUsedAt` — which is truthful, the key was just used.

Validation is a convenience for the user, not a gate you enforce against them: when they explicitly choose write-anyway, write, and mark the report `(not validated)`.
