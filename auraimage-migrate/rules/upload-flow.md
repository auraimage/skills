# upload-flow

How to add new images to AuraImage at runtime (user uploads, CMS integrations, etc.).

## Why Server-Side Signing

AuraImage uses HMAC-signed upload tokens. Your secret key **never leaves the server** — the client receives a short-lived token and uploads directly to the CDN.

## Server-Side: Generate a Token

Using the `@auraimage/sdk`:

```ts
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });

// In your API route / server action:
export async function POST() {
  const token = await aura.signUpload({
    slug: 'my-project',
    userId: 'usr_xxx',
    projectId: 'proj_xxx',
    tier: 'hacker',          // 'hacker' | 'pro' | 'startup'
    maxSize: '5mb',           // optional, default 5mb
    allowedTypes: ['image/*'] // optional, default image/*
  });

  return Response.json({ token });
}
```

## Client-Side: Upload the File

```ts
async function uploadImage(file: File): Promise<string> {
  // 1. Get a signed token from your server
  const { token } = await fetch('/api/upload-token').then(r => r.json());

  // 2. Upload directly to AuraImage
  const form = new FormData();
  form.append('file', file);
  form.append('filename', file.name);

  const res = await fetch('https://api.auraimage.ai/v1/upload', {
    method: 'POST',
    headers: { 'X-Aura-Signature': token },
    body: form,
  });

  const { url } = await res.json();
  // url = "https://auraimage.ai/my-project/photo.jpg"
  return url;
}
```

## What the Upload Response Contains

```ts
interface UploadResult {
  url: string;       // CDN URL — store this in your database
  blurhash: string;  // BlurHash for placeholder (e.g. with blurhash package)
  width: number;     // Original image width in px
  height: number;    // Original image height in px
  format: string;    // Detected format of the original (e.g. "jpeg")
  size: number;      // File size in bytes
}
```

## Storing the Result

Store `url`, `width`, `height`, and optionally `blurhash` in your database. You'll need `width` and `height` to set the correct dimensions on `<img>` elements (prevents CLS).

## Token Expiry

Tokens expire in 1 hour by default. Generate a fresh token per upload session — do not reuse tokens across sessions or store them client-side between page loads.
