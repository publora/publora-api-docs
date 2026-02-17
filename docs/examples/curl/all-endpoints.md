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
  "connections": [
    {
      "id": "twitter-123456789",
      "platform": "twitter",
      "name": "@yourhandle",
      "profileUrl": "https://twitter.com/yourhandle"
    },
    {
      "id": "linkedin-ABC123DEF",
      "platform": "linkedin",
      "name": "John Doe",
      "profileUrl": "https://linkedin.com/in/johndoe"
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

### Post with Image URL

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Check out this screenshot!",
    "platforms": ["twitter-123456789", "linkedin-ABC123DEF"],
    "mediaUrls": ["https://example.com/images/screenshot.png"]
  }'
```

### Post with Multiple Images (Carousel)

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "5 tips for better productivity. Swipe through!",
    "platforms": ["instagram-789012345"],
    "mediaUrls": [
      "https://example.com/images/tip1.jpg",
      "https://example.com/images/tip2.jpg",
      "https://example.com/images/tip3.jpg",
      "https://example.com/images/tip4.jpg",
      "https://example.com/images/tip5.jpg"
    ]
  }'
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
    "mediaUrls": ["https://example.com/videos/bts.mp4"],
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
    "mediaUrls": ["https://example.com/videos/tip.mp4"],
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
  "postGroupId": "pg_abc123xyz",
  "posts": [
    {
      "platform": "twitter",
      "platformConnectionId": "twitter-123456789",
      "status": "scheduled"
    }
  ]
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
  "postGroupId": "pg_abc123xyz",
  "content": "Hello from Publora API!",
  "status": "published",
  "scheduledTime": "2026-03-01T14:00:00.000Z",
  "posts": [
    {
      "platform": "twitter",
      "platformConnectionId": "twitter-123456789",
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
  "success": true,
  "message": "Post deleted successfully"
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
    "mimeType": "image/png"
  }'
```

**Response:**
```json
{
  "uploadUrl": "https://s3.amazonaws.com/bucket/path?signature=...",
  "mediaKey": "mk_abc123xyz"
}
```

### Upload File to S3

```bash
curl -X PUT "https://s3.amazonaws.com/bucket/path?signature=..." \
  -H "Content-Type: image/png" \
  --data-binary @screenshot.png
```

### Create Post with Uploaded Media

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Check out this screenshot!",
    "platforms": ["twitter-123456789"],
    "mediaKeys": ["mk_abc123xyz"]
  }'
```

## LinkedIn Statistics

### Get Post Statistics

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformConnectionId": "linkedin-ABC123DEF",
    "postUrn": "urn:li:share:7123456789012345678"
  }'
```

**Response:**
```json
{
  "impressions": 1250,
  "clicks": 45,
  "likes": 28,
  "comments": 5,
  "shares": 3
}
```

### Get Account Statistics

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-account-statistics \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformConnectionId": "linkedin-ABC123DEF"
  }'
```

## LinkedIn Reactions

### Add Reaction

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reactions \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformConnectionId": "linkedin-ABC123DEF",
    "postUrn": "urn:li:share:7123456789012345678",
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
    "platformConnectionId": "linkedin-ABC123DEF",
    "postUrn": "urn:li:share:7123456789012345678"
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
echo "$CONNECTIONS" | jq '.connections[].id'

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
