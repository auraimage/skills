# placeholder-loading

Pick the right `placeholder` strategy for `<AuraImage />` based on the use case.

## How it works

```
Placeholder (paint-fast)
  ↓ blurhash | lqip | empty
Full image (network-dependent)
  ↓ ?w=…&h=…&q=…&fmt=auto&fit=… — crossfades over the placeholder
```

The component renders the placeholder immediately and crossfades the full image in once `<img onLoad>` fires (0.3s opacity transition).

## `placeholder='blurhash'` (default)

The component fetches the stored BlurHash from `/v1/blurhash/<src>`, decodes it client-side into a tiny canvas DataURL, and renders it blurred while the full image loads. ~30-byte hash, one small cacheable request, instant once the response lands.

```tsx
<AuraImage
  src='my-app/hero.jpg'
  alt='Hero'
  width={1200}
  height={800}
  placeholder='blurhash'   // default — can be omitted
/>
```

There is **no** `blurhash` prop on `<AuraImage />`. The component fetches the value itself. That keeps SSR markup tiny and lets you change crops without re-passing the hash.

If the blurhash fetch fails (network error, or the image was uploaded with no hash), the component automatically falls back to `lqip`.

## `placeholder='lqip'`

Skip BlurHash and fetch a 50-pixel-wide low-quality preview (`?lqip=true`) directly. ~1–3 KB, sharper than BlurHash, but one extra HTTP request.

```tsx
<AuraImage
  src='my-app/photo.jpg'
  alt='Photo'
  width={800}
  height={600}
  placeholder='lqip'
/>
```

## `placeholder='empty'`

No placeholder. Container reserves space (no CLS) but stays blank until the full image loads.

```tsx
<AuraImage
  src='my-app/icon.jpg'
  alt='Icon'
  width={48}
  height={48}
  placeholder='empty'
/>
```

## Picking a strategy

| Use case | Recommended config |
|----------|--------------------|
| Hero / LCP image | `priority` + `placeholder='blurhash'` (default) |
| Content images (blog, gallery) | `placeholder='blurhash'` (default) |
| Thumbnails (cards, avatars) | `placeholder='lqip'` — BlurHash overkill at small sizes |
| Decorative / icons | `placeholder='empty'` |

## Reporting LCP

Set `telemetry` on the LCP image to feed your dashboard's LCP charts:

```tsx
<AuraImage
  src='my-app/hero.jpg'
  alt='Hero'
  width={1200}
  height={800}
  priority
  telemetry
/>
```

This wires up a `PerformanceObserver` that POSTs `{ lcp, projectName, pathname }` to `${NEXT_PUBLIC_AURA_CDN_URL}/v1/telemetry`. No PII.
