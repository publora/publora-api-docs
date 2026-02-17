# cURL Reference: All Endpoints

Complete cURL examples for every Publora API endpoint.

## Setup

Set your API key as an environment variable:

```bash
export PUBLORA_API_KEY="YOUR_API_KEY"
```

## List Platform Connections

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

**Response:**
```json
{
  "success": true,
  "connections": [
    {
      "platformId": "twitter-123456789",
      "username": "@yourhandle",
      "displayName": "Your Handle"
    },
    {
      "platformId": "linkedin-ABC123DEF",
      "username": "johndoe",
      "displayName": "John Doe"
    }
  ]
}
```

## Create Post

### Simple Text Post

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Hello from Publora API!",
    "platforms": ["twitter-123456789", "linkedin-ABC123DEF"]
  }'
```

### Scheduled Post

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "This post will go live tomorrow at 2 PM UTC",
    "platforms": ["twitter-123456789"],
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

### Post with Media

```bash
# Step 1: Create the post first
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Check out this screenshot!",
    "platforms": ["twitter-123456789", "linkedin-ABC123DEF"]
  }'
# Note: postGroupId from the response

# Step 2: Get an upload URL
curl -X POST https://api.publora.com/api/v1/get-upload-url \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fileName": "screenshot.png",
    "contentType": "image/png",
    "postGroupId": "POST_GROUP_ID_FROM_STEP_1"
  }'

# Step 3: Upload to the returned URL
curl -X PUT "UPLOAD_URL_FROM_STEP_2" \
  -H "Content-Type: image/png" \
  --data-binary @screenshot.png
```

### Post with Multiple Images (Carousel)

```bash
# For carousel posts, repeat the upload workflow for each image.
# Each upload uses the same postGroupId.
# See the Media Uploads guide for details.
```

### Draft Post

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Work in progress - will publish later",
    "platforms": ["twitter-123456789"],
    "status": "draft"
  }'
```

### Post with Platform Settings

```bash
# Instagram Reel
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Behind the scenes! #buildinpublic",
    "platforms": ["instagram-789012345"],
    "platformSettings": {
      "instagram": {
        "videoType": "REELS"
      }
    }
  }'

# TikTok with settings
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Quick coding tip! #coding #devtips",
    "platforms": ["tiktok-456789012"],
    "platformSettings": {
      "tiktok": {
        "disableDuet": false,
        "disableStitch": false
      }
    }
  }'

# Telegram with parse mode
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "*Bold* and _italic_ text with [link](https://example.com)",
    "platforms": ["telegram-1001234567890"],
    "platformSettings": {
      "telegram": {
        "parseMode": "MarkdownV2"
      }
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "postGroupId": "pg_abc123xyz"
}
```

## Get Post

```bash
curl https://api.publora.com/api/v1/get-post/pg_abc123xyz \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

**Response:**
```json
{
  "success": true,
  "postGroupId": "pg_abc123xyz",
  "content": "Hello from Publora API!",
  "status": "published",
  "scheduledTime": "2026-03-01T14:00:00.000Z",
  "posts": [
    {
      "platform": "twitter",
      "platformId": "twitter-123456789",
      "status": "published",
      "publishedAt": "2026-03-01T14:00:05.123Z",
      "platformPostId": "1234567890123456789",
      "platformPostUrl": "https://twitter.com/yourhandle/status/1234567890123456789"
    }
  ]
}
```

## Update Post

### Reschedule

```bash
curl -X PUT https://api.publora.com/api/v1/update-post/pg_abc123xyz \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "scheduledTime": "2026-03-15T10:00:00.000Z"
  }'
```

### Change to Draft

```bash
curl -X PUT https://api.publora.com/api/v1/update-post/pg_abc123xyz \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "draft"
  }'
```

### Schedule a Draft

```bash
curl -X PUT https://api.publora.com/api/v1/update-post/pg_abc123xyz \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "scheduled",
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

## Delete Post

```bash
curl -X DELETE https://api.publora.com/api/v1/delete-post/pg_abc123xyz \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

**Response:**
```json
{
  "success": true
}
```

## Upload Media

### Get Upload URL

```bash
curl -X POST https://api.publora.com/api/v1/get-upload-url \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fileName": "screenshot.png",
    "contentType": "image/png",
    "postGroupId": "pg_abc123xyz"
  }'
```

**Response:**
```json
{
  "success": true,
  "uploadUrl": "https://s3.amazonaws.com/bucket/path?signature=...",
  "fileUrl": "https://cdn.publora.com/uploads/screenshot.png",
  "mediaId": "abc123"
}
```

### Upload File to S3

```bash
curl -X PUT "https://s3.amazonaws.com/bucket/path?signature=..." \
  -H "Content-Type: image/png" \
  --data-binary @screenshot.png
```

### Attaching Media to Posts

Media files uploaded with a `postGroupId` are automatically attached to that post group. No additional step is needed to link media to a post.

## LinkedIn Statistics

### Get Post Statistics

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformId": "linkedin-ABC123DEF",
    "postedId": "urn:li:share:7123456789012345678",
    "queryTypes": "ALL"
  }'
```

**Response:**
```json
{
  "success": true,
  "metrics": {
    "IMPRESSION": 1250,
    "MEMBERS_REACHED": 680,
    "RESHARE": 3,
    "REACTION": 28,
    "COMMENT": 5
  },
  "cached": false
}
```

### Get Account Statistics

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-account-statistics \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformId": "linkedin-ABC123DEF",
    "queryTypes": "ALL"
  }'
```

## LinkedIn Reactions

### Add Reaction

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reactions \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformId": "linkedin-ABC123DEF",
    "postedId": "urn:li:share:7123456789012345678",
    "reactionType": "LIKE"
  }'
```

Reaction types: `LIKE`, `CELEBRATE`, `SUPPORT`, `LOVE`, `INSIGHTFUL`, `FUNNY`

### Remove Reaction

```bash
curl -X DELETE https://api.publora.com/api/v1/linkedin-reactions \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformId": "linkedin-ABC123DEF",
    "postedId": "urn:li:share:7123456789012345678"
  }'
```

## Bash Script: Full Workflow

```bash
#!/bin/bash

# Full Publora API workflow

API_KEY="YOUR_API_KEY"
BASE_URL="https://api.publora.com/api/v1"

# 1. List connections
echo "=== Getting connections ==="
CONNECTIONS=$(curl -s "$BASE_URL/platform-connections" \
  -H "x-publora-key: $API_KEY")
echo "$CONNECTIONS" | jq '.connections[].platformId'

# 2. Create a post
echo -e "\n=== Creating post ==="
POST_RESULT=$(curl -s -X POST "$BASE_URL/create-post" \
  -H "x-publora-key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Testing via bash script!",
    "platforms": ["twitter-123456789"]
  }')
POST_ID=$(echo "$POST_RESULT" | jq -r '.postGroupId')
echo "Created post: $POST_ID"

# 3. Wait and check status
echo -e "\n=== Checking status ==="
sleep 5
curl -s "$BASE_URL/get-post/$POST_ID" \
  -H "x-publora-key: $API_KEY" | jq '.posts[].status'

echo -e "\n=== Done ==="
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
