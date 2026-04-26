---
name: auraimage-migrate
description: Rules for migrating local or third-party image assets to AuraImage. Use when a user asks to "migrate images", "optimize images", "move assets to AuraImage", or when you detect unoptimized local images (files in /public, /assets, /static, or require('./img')) in a project. Also triggers when you see Cloudinary, Imgix, S3, or plain CDN image URLs that could be replaced.
license: MIT
metadata:
  author: auraimage
  version: "0.1.0"
---

# AuraImage Migration

Step-by-step workflow for migrating image assets to AuraImage — from detection through upload to JSX rewriting.

## When to Apply

Trigger this workflow when:
- A user asks to "migrate", "optimize", or "move" images
- You encounter `<img src="/public/...">` or `require('./image.png')` patterns
- You see Cloudinary (`res.cloudinary.com`), Imgix (`.imgix.net`), or plain S3 (`s3.amazonaws.com`) URLs
- The `audit_lcp` MCP tool reports unoptimized images

## Before You Start: `aura init`

If `aura.config.json` does not exist in the project root, run `aura init` first:

```sh
npx @auraimage/cli init
```

This creates `aura.config.json`:

```json
{
  "projectName": "my-project",
  "projectId": "proj_xxx"
}
```

**Always read `aura.config.json` before migrating.** Read `projectName` from the config — never hardcode it.

## Migration Workflow

### Step 1 — Audit (always run first)

Use the `audit_lcp` MCP tool to scan the project before touching any files:

```
audit_lcp({ directory: "/absolute/path/to/project" })
```

This reports: total unoptimized images, estimated LCP improvement, and the list of files to migrate. **Do not skip this step** — it gives you the exact file list so you don't miss anything or accidentally upload non-image files.

### Step 2 — Dry Run

Run `migrate_assets` with `dryRun: true` first to preview what will be uploaded. Use `projectName` from `aura.config.json`:

```
migrate_assets({
  directory: "/absolute/path/to/project",
  projectName: "my-project",   // from aura.config.json
  userId: "usr_xxx",
  projectId: "proj_xxx",
  dryRun: true
})
```

Show the user the preview and confirm before proceeding.

### Step 3 — Upload

Run `migrate_assets` with `dryRun: false` to perform the actual upload:

```
migrate_assets({
  directory: "/absolute/path/to/project",
  projectName: "my-project",   // from aura.config.json
  userId: "usr_xxx",
  projectId: "proj_xxx",
  dryRun: false
})
```

Capture the returned AuraImage URLs for the rewrite step.

### Step 4 — Rewrite Image References

Replace all original image references in code with AuraImage `<picture>` elements. See `rules/rewrite-jsx.md` for patterns.

### Step 5 — Verify

After rewriting, run `audit_lcp` again on the same directory. The unoptimized image count should be zero.

## Rule Categories

### 1. Detection (HIGH)

- `detect-local-images` — Patterns that indicate migrateable local images.
- `detect-third-party-urls` — Cloudinary, Imgix, S3, and other CDN URLs to replace.

### 2. Rewriting (HIGH)

- `rewrite-jsx` — How to replace `<img>` tags with AuraImage `<picture>` elements.
- `rewrite-css` — How to replace CSS `url()` background images.

### 3. Upload Flow for New Images (MEDIUM)

- `upload-flow` — Server-side signing + client-side PUT for adding new images at runtime.

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/detect-local-images.md
rules/detect-third-party-urls.md
rules/rewrite-jsx.md
rules/upload-flow.md
```
