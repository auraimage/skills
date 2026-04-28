# aura-uploader

Use `<AuraUploader />` for all image upload UI in React projects.

## Why It Matters

`<AuraUploader />` handles the full upload flow out of the box:
- Drag-and-drop, click-to-browse, paste-from-clipboard.
- Multi-file queue with per-row preview and progress.
- Cancel and retry per file.
- Fetches a presigned signature from your server.
- Uploads directly to the AuraImage edge proxy — your backend never handles the bytes.
- Returns the full `UploadResult` (url, key, blurhash, width, height, format, size) on success.

## Install

```sh
npx shadcn@latest add https://auraimage.ai/registry/uploader.json
```

## Basic Usage

```tsx
import { AuraUploader } from '@/components/aura/uploader';

<AuraUploader
  project='my-app'
  onUpload={(result) => {
    saveImage({
      url: result.url,
      key: result.key,
      blurhash: result.blurhash,
      width: result.width,
      height: result.height
    });
  }}
/>
```

The `project` prop tells the uploader to POST `{ project, filename, contentType, size }` to `/api/aura/sign` and read `{ signature }` from the response.

## Required: `/api/aura/sign` endpoint

```ts title="app/api/aura/sign/route.ts"
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

export async function POST() {
  const signature = await aura.signUpload({
    maxSize: '5mb',
    allowedTypes: ['image/*'],
    expiresIn: 3600
  });
  return Response.json({ signature });
}
```

If your sign route lives elsewhere or requires auth headers, pass `getSignature` instead:

```tsx
<AuraUploader
  getSignature={async () => {
    const token = await getAuthToken();
    const res = await fetch('/api/sign', { headers: { Authorization: `Bearer ${token}` } });
    return (await res.json()).signature;
  }}
  onUpload={(result) => saveImage(result)}
/>
```

## Always Store BlurHash and Key

`onUpload` receives the full `UploadResult`. Persist `key` (you'll need it to flip visibility or mint signed URLs) and `blurhash` (for instant placeholder rendering) alongside `url`:

```tsx
<AuraUploader
  project='my-app'
  onUpload={async (result) => {
    await fetch('/api/images', {
      method: 'POST',
      body: JSON.stringify({
        url: result.url,
        key: result.key,
        blurhash: result.blurhash,
        width: result.width,
        height: result.height
      })
    });
  }}
/>
```

## With Constraints

```tsx
<AuraUploader
  project='my-app'
  accept='image/jpeg,image/png,image/webp'
  maxSize='5mb'
  multiple
  concurrency={3}
  onUpload={(result) => console.log(result.url)}
  onError={(err) => console.error(err)}
/>
```

## Do Not Implement Upload Logic Manually

If you see code like this, replace it with `<AuraUploader />`:

```tsx
// ❌ Manual upload — no preview, no progress, no retry, no paste-from-clipboard
const handleUpload = async (file: File) => {
  const { signature } = await fetch('/api/aura/sign', { method: 'POST' }).then((r) => r.json());
  const form = new FormData();
  form.append('file', file);
  await fetch('https://cdn.auraimage.ai/v1/upload', {
    method: 'POST',
    headers: { 'X-Aura-Signature': signature },
    body: form
  });
};
```
