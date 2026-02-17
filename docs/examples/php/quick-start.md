# PHP Quick Start

Integrate Publora API with PHP using cURL and modern PHP practices.

## Requirements

- PHP 7.4+ (8.0+ recommended)
- cURL extension enabled

## Basic Client Class

```php
<?php
// src/PubloraClient.php

class PubloraException extends Exception
{
    public int $statusCode;
    public array $body;

    public function __construct(int $statusCode, string $message, array $body = [])
    {
        parent::__construct($message);
        $this->statusCode = $statusCode;
        $this->body = $body;
    }
}

class PubloraClient
{
    private const BASE_URL = 'https://api.publora.com/api/v1';

    private string $apiKey;
    private ?string $userId;

    public function __construct(string $apiKey, ?string $userId = null)
    {
        $this->apiKey = $apiKey;
        $this->userId = $userId;
    }

    private function request(string $method, string $endpoint, ?array $data = null): array
    {
        $url = self::BASE_URL . $endpoint;

        $headers = [
            'Content-Type: application/json',
            'x-publora-key: ' . $this->apiKey,
        ];

        if ($this->userId) {
            $headers[] = 'x-publora-user-id: ' . $this->userId;
        }

        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL => $url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_CUSTOMREQUEST => $method,
            CURLOPT_HTTPHEADER => $headers,
            CURLOPT_TIMEOUT => 30,
        ]);

        if ($data !== null) {
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
        }

        $response = curl_exec($ch);
        $statusCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $error = curl_error($ch);
        curl_close($ch);

        if ($error) {
            throw new PubloraException(0, 'cURL error: ' . $error);
        }

        $body = json_decode($response, true) ?? [];

        if ($statusCode >= 400) {
            $message = $body['error'] ?? $body['message'] ?? 'Unknown API error';
            throw new PubloraException($statusCode, $message, $body);
        }

        return $body;
    }

    public function getConnections(): array
    {
        $result = $this->request('GET', '/platform-connections');
        return $result['connections'] ?? [];
    }

    public function createPost(array $data): array
    {
        return $this->request('POST', '/create-post', $data);
    }

    public function getPost(string $postGroupId): array
    {
        return $this->request('GET', '/get-post/' . $postGroupId);
    }

    public function updatePost(string $postGroupId, array $updates): array
    {
        return $this->request('PUT', '/update-post/' . $postGroupId, $updates);
    }

    public function deletePost(string $postGroupId): array
    {
        return $this->request('DELETE', '/delete-post/' . $postGroupId);
    }

    public function getUploadUrl(string $fileName, string $contentType, string $postGroupId): array
    {
        return $this->request('POST', '/get-upload-url', [
            'fileName' => $fileName,
            'contentType' => $contentType,
            'postGroupId' => $postGroupId,
        ]);
    }

    public function getLinkedInPostStats(string $platformId, string $postedId): array
    {
        return $this->request('POST', '/linkedin-post-statistics', [
            'platformId' => $platformId,
            'postedId' => $postedId,
            'queryTypes' => 'ALL',
        ]);
    }

    public function getLinkedInAccountStats(string $platformId): array
    {
        return $this->request('POST', '/linkedin-account-statistics', [
            'platformId' => $platformId,
        ]);
    }
}
```

## Usage Examples

### Initialize Client

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);
```

### List Platform Connections

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

try {
    $connections = $client->getConnections();

    echo "Found " . count($connections) . " connected accounts:\n";
    foreach ($connections as $conn) {
        echo "  - {$conn['platform']}: {$conn['username']} ({$conn['platformId']})\n";
    }
} catch (PubloraException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### Create a Simple Post

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

try {
    $response = $client->createPost([
        'content' => 'Hello from PHP! Posting with Publora API.',
        'platforms' => ['twitter-123456789', 'linkedin-ABC123DEF'],
    ]);

    echo "Post created: {$response['postGroupId']}\n";
    foreach ($response['posts'] as $post) {
        echo "  - {$post['platform']}: {$post['status']}\n";
    }
} catch (PubloraException $e) {
    echo "Failed to create post: " . $e->getMessage() . "\n";
}
```

### Schedule a Post

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

// Schedule for tomorrow at 10 AM UTC
$scheduledTime = (new DateTime('+1 day'))
    ->setTime(10, 0, 0)
    ->format(DateTime::ATOM);

try {
    $response = $client->createPost([
        'content' => 'This post will go live tomorrow at 10 AM UTC!',
        'platforms' => ['twitter-123456789'],
        'scheduledTime' => $scheduledTime,
    ]);

    echo "Post scheduled: {$response['postGroupId']}\n";
    echo "Will publish at: $scheduledTime\n";
} catch (PubloraException $e) {
    echo "Failed to schedule: " . $e->getMessage() . "\n";
}
```

### Schedule Multiple Posts (Week of Content)

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

$posts = [
    ['content' => 'Monday motivation: Start your week with purpose!', 'dayOffset' => 0],
    ['content' => 'Tech tip Tuesday: Always version your APIs.', 'dayOffset' => 1],
    ['content' => 'Wednesday wisdom: Ship fast, iterate faster.', 'dayOffset' => 2],
    ['content' => 'Throwback Thursday: How we grew to 10K users.', 'dayOffset' => 3],
    ['content' => 'Feature Friday: Check out our new dashboard!', 'dayOffset' => 4],
];

$baseTime = new DateTime('tomorrow 09:00:00', new DateTimeZone('UTC'));

foreach ($posts as $post) {
    $scheduledTime = (clone $baseTime)
        ->modify("+{$post['dayOffset']} days")
        ->format(DateTime::ATOM);

    try {
        $response = $client->createPost([
            'content' => $post['content'],
            'platforms' => ['twitter-123456789', 'linkedin-ABC123DEF'],
            'scheduledTime' => $scheduledTime,
        ]);

        echo "Scheduled: " . substr($post['content'], 0, 30) . "... -> {$response['postGroupId']}\n";
        usleep(200000); // 200ms rate limiting
    } catch (PubloraException $e) {
        echo "Failed: " . $e->getMessage() . "\n";
    }
}
```

### Create a Post (Media Auto-Attaches via postGroupId)

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

try {
    $response = $client->createPost([
        'content' => 'Check out this amazing screenshot!',
        'platforms' => ['twitter-123456789', 'linkedin-ABC123DEF'],
    ]);

    echo "Post created: {$response['postGroupId']}\n";
} catch (PubloraException $e) {
    echo "Failed: " . $e->getMessage() . "\n";
}
```

### Post with Platform-Specific Settings

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

// Instagram Reel (upload media first, it auto-attaches via postGroupId)
try {
    $response = $client->createPost([
        'content' => 'Behind the scenes! #buildinpublic',
        'platforms' => ['instagram-789012345'],
        'platformSettings' => [
            'instagram' => [
                'videoType' => 'REELS',
            ],
        ],
    ]);

    echo "Instagram Reel scheduled: {$response['postGroupId']}\n";
} catch (PubloraException $e) {
    echo "Failed: " . $e->getMessage() . "\n";
}

// Telegram with Markdown
try {
    $response = $client->createPost([
        'content' => '*Bold* and _italic_ text with [link](https://example.com)',
        'platforms' => ['telegram-1001234567890'],
        'platformSettings' => [
            'telegram' => [
                'parseMode' => 'MarkdownV2',
                'disableWebPagePreview' => false,
            ],
        ],
    ]);

    echo "Telegram message scheduled: {$response['postGroupId']}\n";
} catch (PubloraException $e) {
    echo "Failed: " . $e->getMessage() . "\n";
}
```

### Upload Media and Post

```php
<?php
require_once 'src/PubloraClient.php';

function getMimeType(string $filename): string
{
    $ext = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
    $mimeTypes = [
        'jpg' => 'image/jpeg',
        'jpeg' => 'image/jpeg',
        'png' => 'image/png',
        'gif' => 'image/gif',
        'mp4' => 'video/mp4',
        'webp' => 'image/webp',
    ];
    return $mimeTypes[$ext] ?? 'application/octet-stream';
}

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);
$filePath = './screenshot.png';

try {
    // Step 1: Create post first to get postGroupId
    $response = $client->createPost([
        'content' => 'Check out this screenshot!',
        'platforms' => ['twitter-123456789', 'linkedin-ABC123DEF'],
    ]);

    $postGroupId = $response['postGroupId'];
    echo "Post created: $postGroupId\n";

    // Step 2: Get upload URL (media auto-attaches via postGroupId)
    $fileName = basename($filePath);
    $contentType = getMimeType($fileName);

    $uploadResult = $client->getUploadUrl($fileName, $contentType, $postGroupId);

    // Step 3: Upload file to S3
    $fileContent = file_get_contents($filePath);

    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => $uploadResult['uploadUrl'],
        CURLOPT_CUSTOMREQUEST => 'PUT',
        CURLOPT_POSTFIELDS => $fileContent,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'Content-Type: ' . $contentType,
        ],
    ]);

    $uploadResponse = curl_exec($ch);
    $uploadStatus = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($uploadStatus >= 400) {
        throw new Exception("Upload failed with status $uploadStatus");
    }

    echo "Media uploaded: {$uploadResult['mediaId']}\n";
    echo "File URL: {$uploadResult['fileUrl']}\n";
    echo "Post with uploaded media created: $postGroupId\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### Error Handling

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

try {
    $response = $client->createPost([
        'content' => 'Test post',
        'platforms' => ['twitter-123456789'],
    ]);

    echo "Success: {$response['postGroupId']}\n";
} catch (PubloraException $e) {
    switch ($e->statusCode) {
        case 401:
            echo "Authentication failed. Check your API key.\n";
            break;
        case 403:
            echo "Access denied: " . $e->getMessage() . "\n";
            // Handle subscription or limit issues
            break;
        case 400:
            echo "Bad request: " . $e->getMessage() . "\n";
            // Handle validation errors
            break;
        case 404:
            echo "Not found: " . $e->getMessage() . "\n";
            break;
        case 429:
            echo "Rate limited. Retry after delay.\n";
            break;
        case 500:
            echo "Server error. Retry later.\n";
            break;
        default:
            echo "API error ({$e->statusCode}): " . $e->getMessage() . "\n";
    }
} catch (Exception $e) {
    echo "Network or unexpected error: " . $e->getMessage() . "\n";
}
```

### Monitor Post Status

```php
<?php
require_once 'src/PubloraClient.php';

function waitForPublish(PubloraClient $client, string $postGroupId, int $maxAttempts = 30): array
{
    for ($attempt = 1; $attempt <= $maxAttempts; $attempt++) {
        $postGroup = $client->getPost($postGroupId);
        $status = $postGroup['status'];

        echo "[$attempt/$maxAttempts] Status: $status\n";

        switch ($status) {
            case 'published':
                echo "All platforms published successfully!\n";
                return $postGroup;

            case 'partially_published':
                echo "Partial success:\n";
                foreach ($postGroup['posts'] as $post) {
                    if ($post['status'] === 'failed') {
                        echo "  - {$post['platform']}: FAILED - {$post['error']}\n";
                    } else {
                        echo "  - {$post['platform']}: {$post['status']}\n";
                    }
                }
                return $postGroup;

            case 'failed':
                echo "All platforms failed:\n";
                foreach ($postGroup['posts'] as $post) {
                    echo "  - {$post['platform']}: {$post['error']}\n";
                }
                return $postGroup;
        }

        sleep(10);
    }

    throw new Exception("Timeout waiting for post $postGroupId to publish");
}

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

try {
    $response = $client->createPost([
        'content' => 'Publishing now!',
        'platforms' => ['twitter-123456789'],
    ]);

    $postGroup = waitForPublish($client, $response['postGroupId']);
    echo "Final status: {$postGroup['status']}\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### Import from CSV

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

$csvFile = 'posts.csv';
$handle = fopen($csvFile, 'r');

// Skip header
fgetcsv($handle);

$results = [];

while (($row = fgetcsv($handle)) !== false) {
    [$content, $platforms, $scheduledTime] = $row;

    $platformIds = array_map('trim', explode(';', $platforms));

    $postData = [
        'content' => $content,
        'platforms' => $platformIds,
        'scheduledTime' => $scheduledTime,
    ];

    try {
        $response = $client->createPost($postData);
        $results[] = [
            'content' => substr($content, 0, 30) . '...',
            'success' => true,
            'postGroupId' => $response['postGroupId'],
        ];
        echo "✓ " . substr($content, 0, 40) . "...\n";
    } catch (PubloraException $e) {
        $results[] = [
            'content' => substr($content, 0, 30) . '...',
            'success' => false,
            'error' => $e->getMessage(),
        ];
        echo "✗ " . substr($content, 0, 40) . "... - {$e->getMessage()}\n";
    }

    usleep(200000); // Rate limiting
}

fclose($handle);

$successful = count(array_filter($results, fn($r) => $r['success']));
echo "\nImported $successful/" . count($results) . " posts\n";
```

### LinkedIn Analytics

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

$platformId = 'linkedin-ABC123DEF';
$postedIds = [
    'urn:li:share:7123456789012345678',
    'urn:li:share:7234567890123456789',
];

echo "=== LinkedIn Analytics Report ===\n\n";

$totalImpressions = 0;
$totalEngagement = 0;

foreach ($postedIds as $postedId) {
    try {
        $result = $client->getLinkedInPostStats($platformId, $postedId);

        $metrics = $result['metrics'];
        $engagement = $metrics['REACTION'] + $metrics['COMMENT'] + $metrics['RESHARE'];
        $totalImpressions += $metrics['IMPRESSION'];
        $totalEngagement += $engagement;

        echo "Post: $postedId\n";
        echo "  Impressions: " . number_format($metrics['IMPRESSION']) . "\n";
        echo "  Members Reached: " . number_format($metrics['MEMBERS_REACHED']) . "\n";
        echo "  Engagement: $engagement ({$metrics['REACTION']} reactions, {$metrics['COMMENT']} comments, {$metrics['RESHARE']} reshares)\n\n";
    } catch (PubloraException $e) {
        echo "Post: $postedId - Error: {$e->getMessage()}\n\n";
    }
}

echo "=== TOTALS ===\n";
echo "Total Impressions: " . number_format($totalImpressions) . "\n";
echo "Total Engagement: $totalEngagement\n";
if ($totalImpressions > 0) {
    echo "Avg Engagement Rate: " . number_format(($totalEngagement / $totalImpressions) * 100, 2) . "%\n";
}
```

### Batch Delete Posts

```php
<?php
require_once 'src/PubloraClient.php';

$client = new PubloraClient($_ENV['PUBLORA_API_KEY']);

$postGroupIds = ['pg_abc123', 'pg_def456', 'pg_ghi789'];

$results = [];

foreach ($postGroupIds as $id) {
    try {
        $client->deletePost($id);
        $results[] = ['id' => $id, 'success' => true];
        echo "✓ Deleted $id\n";
    } catch (PubloraException $e) {
        $results[] = ['id' => $id, 'success' => false, 'error' => $e->getMessage()];
        echo "✗ Failed to delete $id: {$e->getMessage()}\n";
    }

    usleep(100000); // 100ms rate limiting
}

$successful = count(array_filter($results, fn($r) => $r['success']));
echo "\nDeleted $successful/" . count($postGroupIds) . " posts\n";
```

### Laravel Integration

```php
<?php
// config/services.php
return [
    // ...
    'publora' => [
        'api_key' => env('PUBLORA_API_KEY'),
    ],
];

// app/Services/PubloraService.php
namespace App\Services;

use Illuminate\Support\Facades\Http;

class PubloraService
{
    private const BASE_URL = 'https://api.publora.com/api/v1';
    private string $apiKey;

    public function __construct()
    {
        $this->apiKey = config('services.publora.api_key');
    }

    public function createPost(string $content, array $platforms, ?string $scheduledTime = null): array
    {
        $response = Http::withHeaders([
            'x-publora-key' => $this->apiKey,
        ])->post(self::BASE_URL . '/create-post', [
            'content' => $content,
            'platforms' => $platforms,
            'scheduledTime' => $scheduledTime,
        ]);

        if ($response->failed()) {
            throw new \Exception($response->json('error') ?? 'Failed to create post');
        }

        return $response->json();
    }

    public function getConnections(): array
    {
        $response = Http::withHeaders([
            'x-publora-key' => $this->apiKey,
        ])->get(self::BASE_URL . '/platform-connections');

        return $response->json('connections', []);
    }
}

// Usage in controller
use App\Services\PubloraService;

class SocialController extends Controller
{
    public function schedulePost(Request $request, PubloraService $publora)
    {
        $validated = $request->validate([
            'content' => 'required|string|max:3000',
            'platforms' => 'required|array',
            'scheduled_time' => 'nullable|date|after:now',
        ]);

        $result = $publora->createPost(
            $validated['content'],
            $validated['platforms'],
            $validated['scheduled_time'] ?? null
        );

        return response()->json([
            'success' => true,
            'post_group_id' => $result['postGroupId'],
        ]);
    }
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
