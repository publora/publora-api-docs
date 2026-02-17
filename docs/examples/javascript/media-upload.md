# JavaScript Media Upload Examples

Upload images and videos to Publora using the pre-signed URL workflow.

## Upload a Single Image

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

async function uploadImage(filePath, fileName, mimeType) {
  // Step 1: Get pre-signed upload URL
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({ fileName, mimeType })
  });

  const { uploadUrl, mediaKey } = await urlResponse.json();

  // Step 2: Upload file to S3
  const fileBuffer = await fs.promises.readFile(filePath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': mimeType },
    body: fileBuffer
  });

  console.log('Uploaded! Media key:', mediaKey);
  return mediaKey;
}

// Usage
const mediaKey = await uploadImage(
  './product-screenshot.png',
  'product-screenshot.png',
  'image/png'
);
```

## Post with Uploaded Image

```javascript
async function postWithImage(content, platforms, imagePath) {
  // Upload the image first
  const mediaKey = await uploadImage(
    imagePath,
    'post-image.jpg',
    'image/jpeg'
  );

  // Create post with media
  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({
      content,
      platforms,
      mediaKeys: [mediaKey]
    })
  });

  return response.json();
}

// Usage
await postWithImage(
  'Check out our new feature!',
  ['twitter-123456', 'linkedin-ABC123'],
  './feature-screenshot.jpg'
);
```

## Upload Multiple Images (Carousel)

```javascript
async function uploadMultipleImages(files) {
  const mediaKeys = [];

  for (const file of files) {
    const mediaKey = await uploadImage(
      file.path,
      file.name,
      file.mimeType
    );
    mediaKeys.push(mediaKey);
  }

  return mediaKeys;
}

async function postCarousel(content, platforms, imagePaths) {
  // Prepare file info
  const files = imagePaths.map((path, i) => ({
    path,
    name: `image-${i + 1}.jpg`,
    mimeType: 'image/jpeg'
  }));

  // Upload all images
  const mediaKeys = await uploadMultipleImages(files);

  // Create carousel post
  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({
      content,
      platforms,
      mediaKeys // Up to 4 for most platforms, 10 for Instagram
    })
  });

  return response.json();
}

// Usage: Post a carousel to Instagram
await postCarousel(
  '5 tips for better productivity. Swipe through!',
  ['instagram-789012'],
  ['./tip1.jpg', './tip2.jpg', './tip3.jpg', './tip4.jpg', './tip5.jpg']
);
```

## Upload Video

```javascript
async function uploadVideo(videoPath, fileName) {
  // Step 1: Get pre-signed URL for video
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({
      fileName,
      mimeType: 'video/mp4'
    })
  });

  const { uploadUrl, mediaKey } = await urlResponse.json();

  // Step 2: Upload video file
  const videoBuffer = await fs.promises.readFile(videoPath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'video/mp4' },
    body: videoBuffer
  });

  return mediaKey;
}

async function postVideo(content, platforms, videoPath, platformSettings = {}) {
  const mediaKey = await uploadVideo(videoPath, 'video.mp4');

  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({
      content,
      platforms,
      mediaKeys: [mediaKey],
      platformSettings
    })
  });

  return response.json();
}

// Post video as Instagram Reel
await postVideo(
  'Behind the scenes of building our product! #buildinpublic',
  ['instagram-789012'],
  './bts-video.mp4',
  {
    instagram: { videoType: 'REELS' }
  }
);

// Post video to TikTok
await postVideo(
  'Quick tip for developers #coding #devtips',
  ['tiktok-345678'],
  './tip-video.mp4',
  {
    tiktok: { disableDuet: false, disableStitch: false }
  }
);
```

## Upload from URL (Remote Image)

```javascript
async function postWithRemoteImage(content, platforms, imageUrl) {
  // Publora also accepts mediaUrls for remote images
  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({
      content,
      platforms,
      mediaUrls: [imageUrl] // Direct URL to image
    })
  });

  return response.json();
}

// Usage with remote image URL
await postWithRemoteImage(
  'Check out this chart from our analytics!',
  ['twitter-123456', 'linkedin-ABC123'],
  'https://example.com/charts/monthly-growth.png'
);
```

## Upload with Progress (Node.js)

```javascript
const fs = require('fs');
const { Readable } = require('stream');

async function uploadWithProgress(filePath, fileName, mimeType) {
  // Get upload URL
  const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify({ fileName, mimeType })
  });

  const { uploadUrl, mediaKey } = await urlResponse.json();

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
      'Content-Type': mimeType,
      'Content-Length': fileSize.toString()
    },
    body: fileBuffer
  });

  console.log('\nUpload complete!');
  return mediaKey;
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
