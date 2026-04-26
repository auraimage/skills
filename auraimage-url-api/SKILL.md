---
name: auraimage-url-api
description: Rules for constructing AuraImage transformation URLs and responsive image elements. Use when writing or reviewing any code that displays images — <img> tags, CSS background-image, Next.js Image components, or image-related utility functions. Triggers on tasks involving image display, optimization, or responsive layouts.
license: MIT
metadata:
  author: auraimage
  version: "0.1.0"
---

# AuraImage URL API

Guidelines for constructing AuraImage CDN URLs, generating responsive `<picture>` elements, and selecting the right transformation parameters.

## When to Apply

Reference these rules when:
- Writing or reviewing any `<img>`, `<picture>`, or CSS `url()` that serves images
- Implementing responsive image layouts
- Choosing image formats or quality settings
- Building image-heavy pages (galleries, product listings, hero sections)

## URL Format

```
https://auraimage.ai/{projectName}/{filename}?{params}
```

- `projectName` — your project projectName (from the AuraImage dashboard)
- `filename` — the uploaded filename (e.g. `photo.jpg`)
- `params` — transformation query parameters (see below)

## Transformation Parameters

| Param | Type | Description |
|-------|------|-------------|
| `w` | integer | Output width in px. Always set this — never serve full-resolution images. |
| `h` | integer | Output height in px. Optional; omit to preserve aspect ratio. |
| `q` | 1–100 | Quality. Default: `80`. Use `60`–`75` for photos, `90` for logos/UI. |
| `fmt` | `avif` \| `webp` \| `jpeg` | Explicit format override. Omit to let AuraImage auto-negotiate via `Accept` header. |
| `fit` | `cover` \| `face` \| `auto` \| `contain` | Crop mode. Default: `cover`. |
| `blur` | `true` | Returns a blurred placeholder version of the image from the CDN. Use as an `<img>` `src` while the full image loads. Prefer the stored `blurhash` string for LCP images (zero network request). |

## Rule Categories

### 1. Always Set a Width (CRITICAL)

- `url-width-required` — Every AuraImage URL must include `?w=`. Serving at original resolution defeats CDN caching and LCP.

### 2. Responsive Images (HIGH)

- `responsive-picture-element` — Use `<picture>` with AVIF → WebP fallback srcsets for any content image wider than 200 px.
- `responsive-srcset-breakpoints` — Use at minimum three widths: `400`, `800`, `1200`. Add `2x` variants (`800`, `1600`, `2400`) for hero images.

### 3. Format Selection (HIGH)

- `format-auto-negotiate` — Omit `fmt` in production; the CDN returns the best format the browser supports (AVIF → WebP → JPEG).
- `format-explicit-override` — Only set `fmt` when you need a specific format for a non-browser consumer (e.g. an `<og:image>` tag must be JPEG/PNG, so set `fmt=jpeg`).

### 4. Fit Modes (MEDIUM)

- `fit-cover-default` — Use `fit=cover` for thumbnails and cards where the container has a fixed aspect ratio.
- `fit-contain-letterbox` — Use `fit=contain` for logos and product images where cropping is unacceptable.
- `fit-face-portraits` — Use `fit=face` for user avatars and portrait photos to keep faces centered.
- `fit-auto-saliency` — Use `fit=auto` for editorial images where the subject is unpredictable.

### 5. Quality Presets (MEDIUM)

- `quality-photo` — Use `q=75` for photographs. Visually lossless at a fraction of the size.
- `quality-ui` — Use `q=90` for UI assets (icons, logos, illustrations) where detail matters.
- `quality-thumbnail` — Use `q=60` for thumbnails ≤ 200 px wide.

## React / Next.js Projects

In React or Next.js projects, **prefer `<AuraImage />` over writing raw `<picture>` elements**. The component handles Triple-Stage Loading (BlurHash → LQIP → full-resolution image), correct `sizes`, and dev/prod projectName switching automatically.

Install: `npx shadcn@latest add https://auraimage.ai/registry/image.json`

See the `auraimage-react` skill for full component documentation. Use the raw `<picture>` patterns below for plain HTML, email templates, Open Graph tags, and CSS backgrounds.

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/url-construction.md
rules/responsive-picture.md
rules/format-selection.md
rules/fit-modes.md
```

## Quick Reference: Responsive Picture Element

```tsx
<picture>
  <source
    type="image/avif"
    srcSet="
      https://auraimage.ai/{projectName}/{file}?w=400&fmt=avif 400w,
      https://auraimage.ai/{projectName}/{file}?w=800&fmt=avif 800w,
      https://auraimage.ai/{projectName}/{file}?w=1200&fmt=avif 1200w
    "
  />
  <source
    type="image/webp"
    srcSet="
      https://auraimage.ai/{projectName}/{file}?w=400&fmt=webp 400w,
      https://auraimage.ai/{projectName}/{file}?w=800&fmt=webp 800w,
      https://auraimage.ai/{projectName}/{file}?w=1200&fmt=webp 1200w
    "
  />
  <img
    src="https://auraimage.ai/{projectName}/{file}?w=1200"
    alt="..."
    width={1200}
    loading="lazy"
  />
</picture>
```

For Next.js, prefer the MCP tool `generate_responsive_tag` which produces this automatically.
