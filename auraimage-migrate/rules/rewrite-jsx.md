# rewrite-jsx

Replace local or third-party `<img>` tags with AuraImage `<picture>` elements after migration.

## Bare `<img>` → `<picture>`

**Before:**
```tsx
<img src="/public/hero.jpg" alt="Hero" className="w-full" />
```

**After:**
```tsx
<picture>
  <source
    type="image/jxl"
    srcSet="
      https://auraimage.io/{slug}/hero.jpg?w=400&fmt=jxl 400w,
      https://auraimage.io/{slug}/hero.jpg?w=800&fmt=jxl 800w,
      https://auraimage.io/{slug}/hero.jpg?w=1200&fmt=jxl 1200w
    "
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px"
  />
  <source
    type="image/avif"
    srcSet="
      https://auraimage.io/{slug}/hero.jpg?w=400&fmt=avif 400w,
      https://auraimage.io/{slug}/hero.jpg?w=800&fmt=avif 800w,
      https://auraimage.io/{slug}/hero.jpg?w=1200&fmt=avif 1200w
    "
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px"
  />
  <img
    src="https://auraimage.io/{slug}/hero.jpg?w=1200"
    alt="Hero"
    className="w-full"
    width={1200}
    height={800}
    loading="eager"
    fetchpriority="high"
  />
</picture>
```

## next/image → `<picture>`

**Before:**
```tsx
import Image from 'next/image';
import heroImg from '../../public/hero.jpg';

<Image src={heroImg} alt="Hero" fill />
```

**After:** Use the raw `<picture>` pattern above (AuraImage handles resizing and format negotiation, so `next/image` adds no value).

## Thumbnail / Card Image

**Before:**
```tsx
<img src="/public/product.jpg" alt="Product" className="aspect-square object-cover" />
```

**After:**
```tsx
<picture>
  <source type="image/jxl" srcSet="https://auraimage.io/{slug}/product.jpg?w=400&h=400&fit=cover&fmt=jxl 400w" />
  <source type="image/avif" srcSet="https://auraimage.io/{slug}/product.jpg?w=400&h=400&fit=cover&fmt=avif 400w" />
  <img
    src="https://auraimage.io/{slug}/product.jpg?w=400&h=400&fit=cover"
    alt="Product"
    width={400}
    height={400}
    loading="lazy"
  />
</picture>
```

## CSS Background Image

**Before:**
```css
.hero {
  background-image: url('/images/hero.jpg');
}
```

**After:**
```css
.hero {
  background-image: url('https://auraimage.io/{slug}/hero.jpg?w=1920&q=75');
}
```

Note: CSS `background-image` doesn't support `<picture>` format negotiation. Use `q=75` and rely on Accept-header auto-negotiation (no `fmt` param).

## Tips

- Use the `generate_responsive_tag` MCP tool to generate `<picture>` elements automatically rather than writing them by hand.
- Preserve all original `className`, `style`, and event handler props on the inner `<img>`.
- Set `loading="eager"` and `fetchpriority="high"` on the above-the-fold LCP image only; use `loading="lazy"` on all others.
- Always add explicit `width` and `height` to prevent Cumulative Layout Shift (CLS).
