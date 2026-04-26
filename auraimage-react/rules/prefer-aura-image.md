# prefer-aura-image

In React/Next.js projects, always use `<AuraImage />` instead of a raw `<picture>` or `<img>` element.

## Why It Matters

`<AuraImage />` wraps the raw `<picture>` pattern but adds:
- Triple-Stage Loading (BlurHash → LQIP → full image) with zero configuration
- Automatic `sizes` attribute based on `width` prop
- `fetchpriority="high"` when `priority` is set
- Correct `width`/`height` to prevent CLS
- Dev/prod projectName switching from `aura.config.json`

Writing the raw `<picture>` pattern by hand is tedious and easy to get wrong. `<AuraImage />` is the correct abstraction.

## Incorrect (in a React project)

```tsx
// Raw <picture> — verbose, no Triple-Stage Loading, no projectName switching
<picture>
  <source type="image/avif" srcSet="https://auraimage.ai/my-project/hero.jpg?w=1200&fmt=avif" />
  <source type="image/webp" srcSet="https://auraimage.ai/my-project/hero.jpg?w=1200&fmt=webp" />
  <img src="https://auraimage.ai/my-project/hero.jpg?w=1200" alt="Hero" />
</picture>
```

## Correct (in a React project)

```tsx
import { AuraImage } from '@/components/aura/image';

<AuraImage
  projectName="my-project"
  filename="hero.jpg"
  alt="Hero"
  width={1200}
  height={800}
/>
```

## Install if missing

If `@/components/aura/image` doesn't exist in the project:

```sh
npx shadcn@latest add https://auraimage.ai/registry/image.json
```

## Exception: non-React contexts

Use the raw `<picture>` pattern (from `auraimage-url-api`) when:
- Writing plain HTML (no React)
- Writing email templates
- Generating Open Graph meta tags
- Inside CSS `url()` background images
