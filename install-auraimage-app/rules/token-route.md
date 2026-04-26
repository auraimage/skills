# Token route scaffold

Step 4e of the SKILL.md flow. Creates a server-side endpoint that signs upload tokens for `<AuraUploader />`.

## When to scaffold

Only when **all** of these are true:

- A backend was detected (discovery dimension 4).
- The uploader component was installed in step 4d.
- The target file does not already exist (do not overwrite — diff and ask).

If a non-Next backend was detected (Express / Hono / Fastify), do **not** auto-write — surface the framework-appropriate snippet in the final report's "Next" section. The mount path is project-specific.

## Next.js — App Router

Path: `app/api/upload-token/route.ts` (or `.js` if no TypeScript).

```ts
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });

export async function POST() {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',
    tier: 'hacker',
  });
  return Response.json({ token });
}
```

Print this comment in the final report — the placeholders are intentional:

> The route uses placeholder `userId` / `tier`. Wire these to your auth context before going to production.

## Next.js — Pages Router

Path: `pages/api/upload-token.ts`.

```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });

export default async function handler(_req: NextApiRequest, res: NextApiResponse) {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',
    tier: 'hacker',
  });
  res.status(200).json({ token });
}
```

## Express (snippet for report only — do not auto-write)

```ts
import express from 'express';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });
const app = express();

app.post('/api/upload-token', async (_req, res) => {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',
    tier: 'hacker',
  });
  res.json({ token });
});
```

## Hono (snippet for report only)

```ts
import { Hono } from 'hono';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });
const app = new Hono();

app.post('/api/upload-token', async (c) => {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',
    tier: 'hacker',
  });
  return c.json({ token });
});

export default app;
```

## Fastify (snippet for report only)

```ts
import Fastify from 'fastify';
import { AuraImage } from '@auraimage/sdk';

const aura = new AuraImage({ secretKey: process.env.AURA_SECRET_KEY! });
const app = Fastify();

app.post('/api/upload-token', async () => {
  const token = await aura.signUpload({
    projectName: process.env.NEXT_PUBLIC_AURA_PROJECT_NAME!,
    userId: 'usr_xxx',
    tier: 'hacker',
  });
  return { token };
});
```

## Existing-file diff

If the target Next.js path already exists, do not overwrite. Show the user a unified diff against the template above and ask:

> `app/api/upload-token/route.ts` already exists. Replace it with the AuraImage scaffold? [y/N]

Default to **N**. The user's existing handler may already be correct or wired to their auth.
