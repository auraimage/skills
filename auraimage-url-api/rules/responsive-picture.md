# responsive-picture-element

Use `<picture>` with JXL → AVIF srcsets for any content image wider than 200 px.

## Why It Matters

A bare `<img>` with a single URL serves the same file to every device. A `<picture>` with srcsets lets browsers on modern devices download JXL (the best compression), while older browsers fall back to AVIF, then WebP, then JPEG — automatically, with zero JS.

## Incorrect

```tsx
// Single URL, no format fallback, no responsive widths
<img src="https://auraimage.ai/slug/photo.jpg?w=1200" alt="Photo" />
```

## Correct

```tsx
<picture>
  <source
    type="image/jxl"
    srcSet="
      https://auraimage.ai/slug/photo.jpg?w=400&fmt=jxl 400w,
      https://auraimage.ai/slug/photo.jpg?w=800&fmt=jxl 800w,
      https://auraimage.ai/slug/photo.jpg?w=1200&fmt=jxl 1200w
    "
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px"
  />
  <source
    type="image/avif"
    srcSet="
      https://auraimage.ai/slug/photo.jpg?w=400&fmt=avif 400w,
      https://auraimage.ai/slug/photo.jpg?w=800&fmt=avif 800w,
      https://auraimage.ai/slug/photo.jpg?w=1200&fmt=avif 1200w
    "
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px"
  />
  <img
    src="https://auraimage.ai/slug/photo.jpg?w=1200"
    alt="Photo"
    width={1200}
    height={800}
    loading="lazy"
  />
</picture>
```

## Notes

- The MCP tool `generate_responsive_tag` generates this pattern automatically — prefer it over writing it by hand.
- Always include `sizes` so the browser picks the right breakpoint.
- Set explicit `width` and `height` on `<img>` to prevent layout shift (CLS).
- Use `loading="eager"` on LCP images (first above-the-fold image); `loading="lazy"` on everything else.
- For hero/LCP images, add `fetchpriority="high"` to the `<img>`.
