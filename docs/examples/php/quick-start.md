# PHP Quick Start

PHP syntax for the five [Core Workflows](../curl/all-endpoints.md). That page is the semantic source of truth.

## Client helper

```php
<?php

$baseUrl = 'https://api.publora.com/api/v1';
$apiKey = getenv('PUBLORA_API_KEY');
$platformId = getenv('PUBLORA_PLATFORM_ID');

function publora(string $method, string $path, ?array $body = null, ?string $key = null): array {
    global $baseUrl, $apiKey;
    $headers = ['Content-Type: application/json', "x-publora-key: {$apiKey}"];
    if ($key !== null) $headers[] = "Idempotency-Key: {$key}";
    $curl = curl_init("{$baseUrl}/{$path}");
    curl_setopt_array($curl, [
        CURLOPT_CUSTOMREQUEST => $method,
        CURLOPT_HTTPHEADER => $headers,
        CURLOPT_POSTFIELDS => $body === null ? null : json_encode($body, JSON_THROW_ON_ERROR),
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_TIMEOUT => 30,
    ]);
    $raw = curl_exec($curl);
    $status = curl_getinfo($curl, CURLINFO_RESPONSE_CODE);
    if ($raw === false || $status < 200 || $status >= 300) {
        throw new RuntimeException("Publora request failed with HTTP {$status}");
    }
    return json_decode($raw, true, flags: JSON_THROW_ON_ERROR);
}
```

## 1. Draft

```php
$draft = publora('POST', 'create-post', [
    'content' => 'Draft from PHP',
    'platforms' => [$platformId],
]);
```

No `scheduledTime` means draft.

## 2. One-shot schedule

```php
$scheduled = publora('POST', 'create-post', [
    'content' => 'Scheduled from PHP',
    'platforms' => [$platformId],
    'scheduledTime' => (new DateTimeImmutable('+5 minutes'))->format(DateTimeInterface::ATOM),
], bin2hex(random_bytes(16)));
```

## 3. Upload and schedule

Follow the [canonical presigned sequence](../curl/all-endpoints.md#3-upload-and-schedule). Upload the bytes to the returned storage URL, then finalize and schedule:

```php
$upload = curl_init($uploadUrl);
curl_setopt_array($upload, [
    CURLOPT_CUSTOMREQUEST => 'PUT',
    CURLOPT_HTTPHEADER => ['Content-Type: image/jpeg'],
    CURLOPT_POSTFIELDS => file_get_contents('photo.jpg'),
    CURLOPT_RETURNTRANSFER => true,
]);
curl_exec($upload);
if (curl_getinfo($upload, CURLINFO_RESPONSE_CODE) >= 300) {
    throw new RuntimeException('Storage upload failed');
}

publora('POST', "complete-media/{$mediaFileId}");
publora('PUT', "update-post/{$postGroupId}", [
    'status' => 'scheduled',
    'scheduledTime' => (new DateTimeImmutable('+5 minutes'))->format(DateTimeInterface::ATOM),
], "schedule-{$postGroupId}");
```

## 4. Update

```php
publora('PUT', "update-post/{$postGroupId}", [
    'status' => 'scheduled',
    'scheduledTime' => (new DateTimeImmutable('+10 minutes'))->format(DateTimeInterface::ATOM),
], "reschedule-{$postGroupId}");
```

## 5. Webhook consumer

```php
<?php
$raw = file_get_contents('php://input');
$expected = hash_hmac('sha256', $raw, getenv('PUBLORA_WEBHOOK_SECRET'));
$received = $_SERVER['HTTP_X_PUBLORA_SIGNATURE'] ?? '';
if (!hash_equals($expected, $received)) {
    http_response_code(401);
    exit;
}
$envelope = json_decode($raw, true, flags: JSON_THROW_ON_ERROR);
http_response_code(204);
```

CSV import, batch orchestration, and LinkedIn analytics remain available through their dedicated guides and the [OpenAPI document](https://docs.publora.com/openapi.yaml); their contract is not copied here. See [Webhooks](../../endpoints/webhooks.md) for delivery behavior.
