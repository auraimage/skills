# fit-modes

Choose the right `fit` parameter based on the image type and container shape.

## Modes

### `fit=cover` (default, all tiers)

Crops to fill the exact `w×h` box, anchored at center. Best for thumbnails, cards, and any fixed-aspect container.

```
https://auraimage.ai/projectName/photo.jpg?w=400&h=400&fit=cover
```

Use when: product cards, blog post thumbnails, avatar fallbacks.

### `fit=contain` (all tiers)

Scales to fit within `w×h` without cropping. Adds letterboxing if aspect ratios differ. Best for logos and product images where cropping is unacceptable.

```
https://auraimage.ai/projectName/logo.png?w=200&h=100&fit=contain
```

Use when: brand logos, product shots on white, technical diagrams.

### `fit=face`

Uses CF-native face detection to keep the subject's face centered in the crop window. Falls back to `cover` if no face is detected.

```
https://auraimage.ai/projectName/portrait.jpg?w=200&h=200&fit=face
```

Use when: user profile photos, team member cards, author avatars.

### `fit=auto`

Uses CF-native saliency detection to find the most important region and center the crop there.

```
https://auraimage.ai/projectName/editorial.jpg?w=600&h=400&fit=auto
```

Use when: editorial/news images, marketing banners, any image where the subject position is unknown.

## Decision Tree

```
Fixed-aspect container?
  ├── Yes, has a face as subject → fit=face
  ├── Yes, unknown subject position → fit=auto
  ├── Yes, subject is a logo/product → fit=contain
  └── Yes, general photo → fit=cover  (default)
  └── No (preserve aspect ratio) → omit h, use only w
```
