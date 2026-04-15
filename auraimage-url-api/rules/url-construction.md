# url-construction

Always include `?w=` on every AuraImage URL.

## Why It Matters

Serving images at original resolution can mean 3–10 MB per request on mobile. Setting `w` tells the CDN to resize before serving, cutting payload by 90%+ and directly improving LCP.

## Incorrect

```tsx
// Missing width — serves full-resolution original
<img src="https://auraimage.io/my-project/hero.jpg" alt="Hero" />
```

## Correct

```tsx
// Width set — CDN returns a resized image
<img src="https://auraimage.io/my-project/hero.jpg?w=1200" alt="Hero" />
```

## With quality

```tsx
// Width + quality — optimal for photos
<img src="https://auraimage.io/my-project/hero.jpg?w=1200&q=75" alt="Hero" />
```

## URL Construction Pattern

```ts
const url = (filename: string, w: number, opts?: { h?: number; q?: number; fmt?: string; fit?: string }) => {
  const params = new URLSearchParams({ w: String(w) });
  if (opts?.h)   params.set('h',   String(opts.h));
  if (opts?.q)   params.set('q',   String(opts.q));
  if (opts?.fmt) params.set('fmt', opts.fmt);
  if (opts?.fit) params.set('fit', opts.fit);
  return `https://auraimage.io/${SLUG}/${filename}?${params}`;
};
```
