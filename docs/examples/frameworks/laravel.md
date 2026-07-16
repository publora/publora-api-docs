# Laravel Integration

This guide covers Laravel-specific configuration, HTTP calls, queues, and webhook handling. Request semantics are owned by [Core Workflows](../curl/all-endpoints.md).

## Configuration

```env
PUBLORA_API_KEY=sk_YOUR_API_KEY
PUBLORA_BASE_URL=https://api.publora.com/api/v1
PUBLORA_WEBHOOK_SECRET=your_webhook_secret
```

```php
// config/services.php
'publora' => [
    'key' => env('PUBLORA_API_KEY'),
    'url' => env('PUBLORA_BASE_URL', 'https://api.publora.com/api/v1'),
    'webhook_secret' => env('PUBLORA_WEBHOOK_SECRET'),
],
```

## Service wrapper

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;

final class Publora
{
    public function request(string $method, string $path, array $body = [], ?string $key = null): array
    {
        $request = Http::baseUrl(config('services.publora.url'))
            ->withHeaders(['x-publora-key' => config('services.publora.key')])
            ->timeout(30);

        if ($key) {
            $request = $request->withHeaders(['Idempotency-Key' => $key]);
        }

        return $request->send($method, $path, ['json' => $body])->throw()->json();
    }
}
```

Use bodies from [Core Workflows](../curl/all-endpoints.md). Do not add client-side platform ID regexes: IDs are opaque connection identifiers after the lowercase platform prefix.

## Queue job

```php
<?php

namespace App\Jobs;

use App\Services\Publora;
use Illuminate\Contracts\Queue\ShouldQueue;

final class SchedulePubloraPost implements ShouldQueue
{
    public function __construct(
        public string $content,
        public array $platforms,
        public string $requestId,
        public string $scheduledTime,
    ) {}

    public function handle(Publora $publora): void
    {
        $publora->request('POST', 'create-post', [
            'content' => $this->content,
            'platforms' => $this->platforms,
            'scheduledTime' => $this->scheduledTime,
        ], "laravel-{$this->requestId}");
    }
}

SchedulePubloraPost::dispatch(
    $content,
    $platforms,
    $requestId,
    now('UTC')->addMinutes(5)->toIso8601String(),
);
```

Compute `scheduledTime` before dispatching the job. Queue retries must reuse both the same timestamp and the same idempotency key; recomputing the timestamp changes the request body and returns `422 IDEMPOTENCY_KEY_CONFLICT`.

## Webhook route

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/webhooks/publora', function (Request $request) {
    $raw = $request->getContent();
    $expected = hash_hmac('sha256', $raw, config('services.publora.webhook_secret'));
    abort_unless(hash_equals($expected, $request->header('x-publora-signature', '')), 401);
    $envelope = json_decode($raw, true, flags: JSON_THROW_ON_ERROR);
    // Dispatch application work, then acknowledge quickly.
    return response()->noContent();
});
```

Exclude this route from CSRF protection as appropriate for your Laravel version, and verify the raw body before parsing. See [Webhooks](../../endpoints/webhooks.md).

## Workflow links

- [Draft and one-shot schedule](../curl/all-endpoints.md#1-create-a-draft)
- [Upload and schedule](../curl/all-endpoints.md#3-upload-and-schedule)
- [Update](../curl/all-endpoints.md#4-update-a-post)
- [Webhook consumer](../curl/all-endpoints.md#5-verify-a-webhook)
- [Complete OpenAPI surface](https://docs.publora.com/openapi.yaml)
