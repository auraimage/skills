# aura-uploader

Use `<AuraUploader />` for all image upload UI in React projects.

## Why It Matters

`<AuraUploader />` handles the full upload flow out of the box:
- Client-side image preview before upload starts
- Progress bar with Radix UI primitives
- Fetches a presigned token from your server
- Uploads directly to the AuraImage edge proxy
- Returns the full `UploadResult` (url, blurhash, width, height) on success
- Built-in error handling and retry logic

## Install

```sh
npx shadcn@latest add https://auraimage.ai/registry/uploader.json
```

## Basic Usage

```tsx
import { AuraUploader } from '@/components/aura/uploader';

<AuraUploader
  tokenEndpoint="/api/upload-token"
  onSuccess={(result) => {
    // Store result.url and result.blurhash in your database
    saveImage({ url: result.url, blurhash: result.blurhash });
  }}
/>
```

## Required: Token Endpoint

The `tokenEndpoint` must be a server-side route that calls `aura.signUpload()`:

```ts
// app/api/upload-token/route.ts
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });

export async function POST() {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',   // from your auth session
    tier: 'hacker',
  });
  return Response.json({ token });
}
```

## Always Store BlurHash

The `onSuccess` callback receives the full `UploadResult`. Always store `blurhash`, `width`, and `height` alongside `url` — you'll need them to render `<AuraImage />` with Triple-Stage Loading:

```tsx
<AuraUploader
  tokenEndpoint="/api/upload-token"
  onSuccess={async (result) => {
    await fetch('/api/images', {
      method: 'POST',
      body: JSON.stringify({
        url: result.url,
        blurhash: result.blurhash,
        width: result.width,
        height: result.height,
      }),
    });
  }}
/>
```

## With Constraints

```tsx
<AuraUploader
  tokenEndpoint="/api/upload-token"
  accept="image/jpeg,image/png,image/webp"
  maxSize="5mb"
  onSuccess={(result) => console.log(result.url)}
  onError={(err) => console.error(err)}
/>
```

## Do Not Implement Upload Logic Manually

If you see code like this, replace it with `<AuraUploader />`:

```tsx
// ❌ Manual upload — verbose, no preview, no error handling
const handleUpload = async (file: File) => {
  const { token } = await fetch('/api/upload-token').then(r => r.json());
  const form = new FormData();
  form.append('file', file);
  await fetch('https://cdn.auraimage.ai/v1/upload', {
    method: 'POST',
    headers: { 'X-Aura-Signature': token },
    body: form,
  });
};
```
