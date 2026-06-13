# Env writes — idempotent upsert

Step 5 of the SKILL.md flow. Runs only after the user confirmed the write screen. The target file and variable names come from `rules/store-selection.md`.

## Upsert algorithm

Read the file (if it exists), process line by line:

```
For each variable to write (secret key, project name):
  if a line starts with `<name>=` (or `export <name>=` in .envrc-style files):
    if the existing value equals the new value → no-op for this variable
    else → replace that line in place
  else:
    append `<name>=<value>` at end of file
    (with a leading newline if the file doesn't end in one)
```

Preserve every other line, comment, and blank line exactly. Never blind-append — that's how re-runs create duplicate keys, and which duplicate wins is loader-dependent.

If the target file doesn't exist, create it containing only the two lines.

Match the file's existing style: if entries use `export NAME=value` (`.envrc`), write that form; if values are quoted throughout, quote consistently.

## Gitignore amendment

If store-selection flagged the target file as uncovered and `.gitignore` exists, append:

```
# AuraImage
.env
```

(using the actual target filename). Skip when already covered. Never create `.gitignore`.

## Hard rules

- The Secret Key value is written to exactly one place: the target file. Not to shell profiles, not to `.env.example`, not echoed into the terminal scrollback via `echo`/`cat` of the full value.
- No client-exposure prefix on the secret, ever (see store-selection — this is the bundle-leak rule).
- On any write error (permissions, read-only fs), stop and report what was and wasn't written. The upsert design means a re-run resumes safely.

## Re-run behavior

Identical values already present → no-op for both variables; the report says "Already configured, nothing changed." Different secret value with user confirmation → in-place replacement (rotation); remind the user to revoke the old key in the dashboard.
