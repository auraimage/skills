# Token route scaffold

Step 4e of the SKILL.md flow. Creates a server-side endpoint that mints upload signatures for `<AuraUploader />`.

The default mount path is `/api/aura/sign` — the path `<AuraUploader project={...} />` POSTs to when no `getSignature` / `signature` prop is set. Returning a different shape or path means the user has to wire it up manually.

## When to scaffold

Only when **all** of these are true:

- A backend was detected (discovery dimension 4).
- The uploader component was installed in step 4d.
- The target file does not already exist (do not overwrite — diff and ask).

If a non-Next backend was detected (Express / Hono / Fastify), do **not** auto-write — surface the framework-appropriate snippet in the final report's "Next" section. The mount path is project-specific.

## Next.js — App Router

Path: `app/api/aura/sign/route.ts` (or `.js` if no TypeScript).

```ts
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

Note in the final report:

> The route mints a 1-hour signature for any caller. Add your auth check (session, JWT, …) before calling `aura.signUpload()` if uploads should be gated.

## Next.js — Pages Router

Path: `pages/api/aura/sign.ts`.

```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

export default async function handler(_req: NextApiRequest, res: NextApiResponse) {
  const signature = await aura.signUpload({
    maxSize: '5mb',
    allowedTypes: ['image/*'],
    expiresIn: 3600
  });
  res.status(200).json({ signature });
}
```

## Express (snippet for report only — do not auto-write)

```ts
import express from 'express';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

const app = express();

app.post('/api/aura/sign', async (_req, res) => {
  const signature = await aura.signUpload({
    maxSize: '5mb',
    allowedTypes: ['image/*'],
    expiresIn: 3600
  });
  res.json({ signature });
});
```

## Hono (snippet for report only)

```ts
import { Hono } from 'hono';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

const app = new Hono();

app.post('/api/aura/sign', async (c) => {
  const signature = await aura.signUpload({
    maxSize: '5mb',
    allowedTypes: ['image/*'],
    expiresIn: 3600
  });
  return c.json({ signature });
});

export default app;
```

## Fastify (snippet for report only)

```ts
import Fastify from 'fastify';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({
  secretKey: process.env.AURA_SECRET_KEY!,
  projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!
});

const app = Fastify();

app.post('/api/aura/sign', async () => {
  const signature = await aura.signUpload({
    maxSize: '5mb',
    allowedTypes: ['image/*'],
    expiresIn: 3600
  });
  return { signature };
});
```

## Existing-file diff

If the target Next.js path already exists, do not overwrite. Show the user a unified diff against the template above and ask:

> `app/api/aura/sign/route.ts` already exists. Replace it with the AuraImage scaffold? [y/N]

Default to **N**. The user's existing handler may already be correct or wired to their auth.
