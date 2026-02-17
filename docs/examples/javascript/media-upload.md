# JavaScript Media Upload Examples

Upload images and videos to Publora using the pre-signed URL workflow.

## How Media Upload Works

1. **Create the post first** using `POST /create-post` (returns a `postGroupId`)
2. **Get a pre-signed upload URL** using `POST /get-upload-url` with the `postGroupId`
3. **Upload the file** directly to the returned S3 URL
4. Media is **automatically attached** to the post via the `postGroupId`

## Upload a Single Image

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';
const fs = require('fs');

async function createPostWithImage(content, platforms, imagePath, fileName, contentType) {
  const headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
  };

  // Step 1: Create the post
  const postResponse = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ content, platforms })
  });

  const { postGroupId } = await postResponse.json();
  console.log('Post created:', postGroupId);

  // Step 2: Get pre-signed upload URL
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ fileName, contentType, postGroupId })
  });

  const { uploadUrl, fileUrl, mediaId } = await urlResponse.json();

  // Step 3: Upload file to S3
  const fileBuffer = await fs.promises.readFile(imagePath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': contentType },
    body: fileBuffer
  });

  console.log('Uploaded! File URL:', fileUrl);
  console.log('Media ID:', mediaId);

  return { postGroupId, fileUrl, mediaId };
}

// Usage
await createPostWithImage(
  'Check out our new feature!',
  ['twitter-123456', 'linkedin-ABC123'],
  './product-screenshot.png',
  'product-screenshot.png',
  'image/png'
);
```

## Upload Multiple Images (Carousel)

```javascript
async function createCarouselPost(content, platforms, images) {
  const headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
  };

  // Step 1: Create the post
  const postResponse = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ content, platforms })
  });

  const { postGroupId } = await postResponse.json();
  console.log('Post created:', postGroupId);

  // Step 2: Upload each image to the same postGroupId
  for (const image of images) {
    const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        fileName: image.name,
        contentType: image.contentType,
        postGroupId
      })
    });

    const { uploadUrl, mediaId } = await urlResponse.json();

    const fileBuffer = await fs.promises.readFile(image.path);
    await fetch(uploadUrl, {
      method: 'PUT',
      headers: { 'Content-Type': image.contentType },
      body: fileBuffer
    });

    console.log(`Uploaded ${image.name} (mediaId: ${mediaId})`);
  }

  return postGroupId;
}

// Usage: Post a carousel to Instagram
await createCarouselPost(
  '5 tips for better productivity. Swipe through!',
  ['instagram-789012'],
  [
    { path: './tip1.jpg', name: 'tip1.jpg', contentType: 'image/jpeg' },
    { path: './tip2.jpg', name: 'tip2.jpg', contentType: 'image/jpeg' },
    { path: './tip3.jpg', name: 'tip3.jpg', contentType: 'image/jpeg' },
    { path: './tip4.jpg', name: 'tip4.jpg', contentType: 'image/jpeg' },
    { path: './tip5.jpg', name: 'tip5.jpg', contentType: 'image/jpeg' }
  ]
);
```

## Upload Video

```javascript
async function createVideoPost(content, platforms, videoPath, fileName) {
  const headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
  };

  // Step 1: Create the post
  const postResponse = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ content, platforms })
  });

  const { postGroupId } = await postResponse.json();

  // Step 2: Get upload URL for video
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      fileName,
      contentType: 'video/mp4',
      postGroupId
    })
  });

  const { uploadUrl, fileUrl, mediaId } = await urlResponse.json();

  // Step 3: Upload video file
  const videoBuffer = await fs.promises.readFile(videoPath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'video/mp4' },
    body: videoBuffer
  });

  console.log('Video uploaded:', fileUrl);
  return { postGroupId, fileUrl, mediaId };
}

// Post video as Instagram Reel
await createVideoPost(
  'Behind the scenes of building our product! #buildinpublic',
  ['instagram-789012'],
  './bts-video.mp4',
  'bts-video.mp4'
);

// Post video to TikTok
await createVideoPost(
  'Quick tip for developers #coding #devtips',
  ['tiktok-345678'],
  './tip-video.mp4',
  'tip-video.mp4'
);
```

## Upload with Progress (Node.js)

```javascript
const fs = require('fs');

async function uploadWithProgress(postGroupId, filePath, fileName, contentType) {
  const headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
  };

  // Get upload URL
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ fileName, contentType, postGroupId })
  });

  const { uploadUrl, mediaId } = await urlResponse.json();

  // Get file stats for progress tracking
  const stats = fs.statSync(filePath);
  const fileSize = stats.size;
  let uploaded = 0;

  // Create read stream
  const fileStream = fs.createReadStream(filePath);

  fileStream.on('data', (chunk) => {
    uploaded += chunk.length;
    const percent = Math.round((uploaded / fileSize) * 100);
    process.stdout.write(`\rUploading: ${percent}%`);
  });

  // Upload with stream
  const fileBuffer = await fs.promises.readFile(filePath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: {
      'Content-Type': contentType,
      'Content-Length': fileSize.toString()
    },
    body: fileBuffer
  });

  console.log('\nUpload complete!');
  return mediaId;
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
