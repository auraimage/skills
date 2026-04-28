# upload-flow

How to add new images to AuraImage at runtime (user uploads, CMS integrations, etc.).

## Why Server-Side Signing

AuraImage uses HMAC-signed upload signatures. Your secret key **never leaves the server** — the client receives a short-lived signature and uploads directly to the CDN.

## Server-Side: Generate a Signature

Using the `@auraimage/sdk`:

```ts
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

// In your API route / server action:
export async function POST() {
  const signature = await aura.signUpload({
    maxSize: '5mb',                // optional, default 5mb
    allowedTypes: ['image/*'],     // optional, default image/*
    expiresIn: 3600                // optional seconds, default 3600
  });

  return Response.json({ signature });
}
```

The convention used by `<AuraUploader />` is to expose this route at `/api/aura/sign`.

## Client-Side: Upload the File

If you're using `<AuraUploader />` (recommended), it does this for you. For a manual flow:

```ts
async function uploadImage(file: File): Promise<string> {
  // 1. Get a signed signature from your server
  const { signature } = await fetch('/api/aura/sign', { method: 'POST' }).then(r => r.json());

  // 2. Upload directly to AuraImage
  const form = new FormData();
  form.append('file', file);
  form.append('filename', file.name);

  const res = await fetch('https://cdn.auraimage.ai/v1/upload', {
    method: 'POST',
    headers: { 'X-Aura-Signature': signature },
    body: form,
  });

  const { url } = await res.json();
  // url = "https://cdn.auraimage.ai/my-project/photo.jpg"
  return url;
}
```

## What the Upload Response Contains

```ts
interface UploadResult {
  url: string;       // CDN URL — store this in your database
  key: string;       // Stable key (projectName/filename) — pass to setVisibility / getSignedUrl
  blurhash: string;  // BlurHash for placeholder rendering
  width: number;     // Original image width in px
  height: number;    // Original image height in px
  format: string;    // Detected format of the original (e.g. "jpeg")
  size: number;      // File size in bytes
}
```

## Storing the Result

Store `url`, `key`, `width`, `height`, and `blurhash` in your database. `key` is what you pass to `aura.setVisibility(key, 'private')` or `aura.getSignedUrl(key)` later. `width`/`height` prevent layout shift on render.

## Signature Expiry

Signatures expire in 1 hour by default (`expiresIn` is in seconds). Generate a fresh signature per upload session — do not reuse signatures across sessions or store them client-side between page loads.
