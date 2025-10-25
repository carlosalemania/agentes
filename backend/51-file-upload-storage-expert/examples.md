# File Upload/Storage Examples

## Example: Complete Upload Flow

```javascript
// Step 1: Get presigned URL
const { url, key } = await fetch('/api/upload/url', {
  method: 'POST',
  body: JSON.stringify({ filename: file.name, contentType: file.type }),
}).then(r => r.json());

// Step 2: Upload directly to S3
await fetch(url, {
  method: 'PUT',
  body: file,
  headers: { 'Content-Type': file.type },
});

// Step 3: Confirm upload
await fetch('/api/upload/confirm', {
  method: 'POST',
  body: JSON.stringify({ key }),
});
```

## Example: Image Processing Pipeline

```javascript
async function processUpload(fileKey) {
  const input = await downloadFromS3(fileKey);

  // Generate variants
  const [optimized, thumbnail, webp] = await Promise.all([
    sharp(input).resize(1200).jpeg({ quality: 80 }).toBuffer(),
    sharp(input).resize(200, 200).jpeg({ quality: 70 }).toBuffer(),
    sharp(input).webp({ quality: 80 }).toBuffer(),
  ]);

  // Upload variants
  await Promise.all([
    uploadToS3(`${fileKey}-optimized.jpg`, optimized),
    uploadToS3(`${fileKey}-thumb.jpg`, thumbnail),
    uploadToS3(`${fileKey}.webp`, webp),
  ]);
}
```
