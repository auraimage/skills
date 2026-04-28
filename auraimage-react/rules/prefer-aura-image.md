# prefer-aura-image

In React/Next.js projects, always use `<AuraImage />` instead of a raw `<picture>` or `<img>` element pointed at the AuraImage CDN.

## Why It Matters

`<AuraImage />` wraps the CDN URL pattern but adds:
- Automatic BlurHash placeholder fetched from the CDN, decoded client-side, crossfaded into the full image.
- LQIP fallback when the BlurHash fetch fails.
- Variant-change handling — the placeholder fades back in when `src` / `width` / `format` / `fit` change, preventing stale frames.
- `fetchpriority="high"` and `loading="eager"` when `priority` is set.
- Correct `width`/`height` so the layout doesn't shift.

Constructing the URL by hand is verbose and skips the placeholder behavior. `<AuraImage />` is the correct abstraction.

## Incorrect (in a React project)

```tsx
// Raw <img> — verbose, no placeholder
<img
  src='https://cdn.auraimage.ai/my-app/hero.jpg?w=1200&q=80&fmt=auto'
  alt='Hero'
  width={1200}
  height={800}
/>
```

## Correct (in a React project)

```tsx
import { AuraImage } from '@/components/aura/image';

<AuraImage
  src='my-app/hero.jpg'
  alt='Hero'
  width={1200}
  height={800}
/>
```

## Install if missing

If `@/components/aura/image` doesn't exist in the project:

```sh
npx shadcn@latest add https://auraimage.ai/registry/image.json
```

The component reads the CDN base URL from `NEXT_PUBLIC_AURA_CDN_URL`.

## Exception: non-React contexts

Use the raw URL pattern (from `auraimage-url-api`) when:
- Writing plain HTML (no React).
- Writing email templates.
- Generating Open Graph meta tags.
- Inside CSS `url()` background images.
