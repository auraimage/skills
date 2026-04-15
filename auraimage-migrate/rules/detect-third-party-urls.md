# detect-third-party-urls

Identify third-party image CDN URLs that can be replaced with AuraImage.

## URL Patterns to Flag

### Cloudinary

```
res.cloudinary.com/{cloud_name}/image/upload/...
```

Cloudinary transformations (e.g. `/w_800,q_75/`) should be stripped and replaced with AuraImage params.

### Imgix

```
{subdomain}.imgix.net/{filename}?w=800&q=75
```

Imgix query params map directly to AuraImage params — `w`, `h`, `q` are identical.

### Amazon S3 / CloudFront

```
{bucket}.s3.amazonaws.com/{key}
{distribution}.cloudfront.net/{key}
```

These serve originals with no transformation. After migrating, replace with AuraImage URLs that include `?w=` and `?q=`.

### Generic unoptimized CDNs

Any `<img src="https://...">` that:
- Lacks width/height params
- Points to a `.jpg`/`.png` larger than 200 KB
- Is not on `auraimage.io`

## Replacement Strategy

1. Download the original from the third-party URL
2. Upload to AuraImage via `migrate_assets` or the upload API
3. Replace the URL in code with the AuraImage URL

## Do Not Replace

- Your own API endpoints that return images (e.g. `/api/avatar`)
- User-generated content URLs stored in a database — replace at the data layer, not in code
- Favicon and web manifest icons
