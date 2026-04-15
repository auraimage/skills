# detect-local-images

Identify local image assets that should be migrated to AuraImage.

## Patterns to Flag

### Static `<img>` pointing to /public or /static

```tsx
// Flag these
<img src="/images/hero.jpg" />
<img src="/public/photo.png" />
<img src="/assets/logo.svg" />
```

### Require/import of local image files

```ts
// Flag these
import heroImg from './hero.jpg';
const logo = require('../assets/logo.png');
```

### next/image with a local file import

```tsx
// Flag — the source is still local; the file should move to AuraImage
import heroImg from '../../public/hero.jpg';
<Image src={heroImg} alt="Hero" />
```

### CSS background-image with relative paths

```css
/* Flag these */
.hero { background-image: url('/images/hero.jpg'); }
.card { background-image: url('../assets/bg.png'); }
```

## Where to Look

When `audit_lcp` is not available (no MCP server), scan these directories manually:

- `public/` — Next.js / Vite static assets
- `assets/` — general asset directories
- `static/` — common in older React setups
- `src/assets/` — bundled assets

File extensions to flag: `.jpg`, `.jpeg`, `.png`, `.webp`, `.gif`, `.tiff`, `.bmp`

SVGs should generally stay as inline SVGs or local files — **do not migrate SVGs** unless they are large bitmap-embedded SVGs.

## Exclusions

Do not flag:
- Images already on `auraimage.io`
- SVG files (unless unusually large)
- `favicon.ico` and web manifest icons
- Images inside `node_modules`
