# triple-stage-loading

Understand the three loading stages so you can configure `<AuraImage />` correctly per use case.

## The Three Stages

```
Stage 1 (Instant, 0ms)
  ↓ BlurHash DataURI decoded in HTML — no network request
Stage 2 (Fast, ~50ms)
  ↓ ?w=50&q=20 LQIP fetched — ~1–3 KB
Stage 3 (Final, network-dependent)
  ↓ Full-resolution JXL/AVIF fades in
```

## Stage 1: BlurHash Placeholder

The `blurhash` string (returned by the upload API) is decoded into a tiny `<canvas>` DataURI and rendered immediately inside the HTML — before any network request. This gives users an instant colorful placeholder with the correct colors and rough shape of the image.

**How to pass it:**

```tsx
<AuraImage
  slug="my-project"
  filename="photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  placeholder="blurhash"
  blurhash="LKO2?U%2Tw=w]~RBVZRi};RPxuwH"
/>
```

**Where to get the `blurhash` string:**

Store it in your database when the user uploads an image:

```ts
const result = await uploadImage(file); // returns UploadResult
await db.images.create({
  url: result.url,
  blurhash: result.blurhash,  // ← store this
  width: result.width,
  height: result.height,
});
```

Then pass it to `<AuraImage />` at render time from your database record.

## Stage 2: LQIP (Low-Quality Image Placeholder)

If no `blurhash` is available, or after Stage 1 while the full image loads, the component fetches a `?w=50&q=20` version of the image. This is ~1–3 KB and loads in milliseconds on any connection.

Use `placeholder="lqip"` to skip Stage 1 and go straight to Stage 2:

```tsx
<AuraImage
  slug="my-project"
  filename="photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  placeholder="lqip"
/>
```

## Stage 3: Full Image

The full-resolution image is fetched using the browser's best supported format (JXL for Chrome 145+/Safari 17+, AVIF for older browsers). It fades in over the placeholder with a CSS `opacity` transition.

## When to Use Which Configuration

| Use Case | Recommended Config |
|----------|--------------------|
| Hero / LCP image | `priority` + `placeholder="blurhash"` + stored `blurhash` |
| Content images (blog, gallery) | `placeholder="blurhash"` + stored `blurhash` |
| Thumbnails (cards, avatars) | `placeholder="lqip"` (blurhash overkill at small sizes) |
| Decorative / non-critical | No placeholder needed |
| Below-the-fold | Default (lazy load, no priority) |

## Stage 1 Without a Stored BlurHash

If you don't have a BlurHash yet (e.g. legacy images), you can generate one on-the-fly using the `blur=true` URL param:

```tsx
// Falls back to a blurred version of the image served by the CDN
<AuraImage
  slug="my-project"
  filename="photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  placeholder="cdn-blur"  // fetches ?blur=true from the CDN
/>
```

Note: `cdn-blur` requires a network request (unlike the zero-request BlurHash DataURI), so prefer the stored `blurhash` string for LCP images.
