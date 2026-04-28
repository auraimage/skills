# format-selection

Omit `fmt` in production to let the CDN auto-negotiate. Only set `fmt` for non-browser consumers.

## Why It Matters

AuraImage reads the `Accept` header and returns the best format the client supports: AVIF → WebP → JPEG. Hardcoding `fmt` in a regular `<img>` tag forces a suboptimal format for many visitors.

## When to Omit `fmt` (Auto-Negotiate)

```tsx
// In a regular <img> or as a fallback src — let the CDN decide
<img src="https://cdn.auraimage.ai/projectName/photo.jpg?w=800" alt="..." />
```

## When to Set `fmt` Explicitly

**Open Graph / social sharing tags** — OG crawlers don't send an `Accept` header and may not support AVIF:

```tsx
// Meta tag — must be JPEG for broad crawler support
<meta property="og:image" content="https://cdn.auraimage.ai/projectName/photo.jpg?w=1200&fmt=jpeg" />
```

**RSS feeds and email** — email clients have poor format support:

```html
<!-- Email template — use JPEG -->
<img src="https://cdn.auraimage.ai/projectName/banner.jpg?w=600&fmt=jpeg" />
```

**`<picture>` srcsets** — here you explicitly list formats per `<source>`, so `fmt` is correct:

```tsx
<source type="image/avif" srcSet="https://cdn.auraimage.ai/projectName/photo.jpg?w=800&fmt=avif 800w" />
<source type="image/webp" srcSet="https://cdn.auraimage.ai/projectName/photo.jpg?w=800&fmt=webp 800w" />
```

## Format Quality Trade-offs

| Format | Size | Support |
|--------|------|---------|
| AVIF | Best | Chrome 85+, Firefox 93+, Safari 16.0+ |
| WebP | Good | All modern browsers |
| JPEG | Baseline | Universal |
