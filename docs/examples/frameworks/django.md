# Django Integration

This guide covers Django-specific configuration and background jobs. Request semantics are single-sourced in [Core Workflows](../curl/all-endpoints.md).

## Installation and settings

```bash
pip install requests celery
```

```python
# settings.py
import os

PUBLORA_API_KEY = os.environ["PUBLORA_API_KEY"]
PUBLORA_BASE_URL = os.getenv("PUBLORA_BASE_URL", "https://api.publora.com/api/v1")
```

Keep the API key server-side. Never expose it in templates or browser JavaScript.

## Small service wrapper

```python
# social/publora.py
import requests
from django.conf import settings

class Publora:
    def request(self, method, path, *, json=None, idempotency_key=None):
        headers = {"x-publora-key": settings.PUBLORA_API_KEY}
        if idempotency_key:
            headers["Idempotency-Key"] = idempotency_key
        response = requests.request(
            method,
            f"{settings.PUBLORA_BASE_URL}/{path}",
            headers=headers,
            json=json,
            timeout=30,
        )
        response.raise_for_status()
        return response.json()
```

Pass the request bodies from [Core Workflows](../curl/all-endpoints.md) to this wrapper. In particular, omitting `scheduledTime` creates a draft; it does not publish.

## Celery scheduling job

Use Celery for application-side orchestration, but send a future ISO timestamp to Publora rather than relying on the task execution time.

```python
# social/tasks.py
from celery import shared_task
from django.utils import timezone
from datetime import timedelta
from .publora import Publora

@shared_task(bind=True, autoretry_for=(Exception,), retry_backoff=True, max_retries=3)
def schedule_post(self, content, platform_ids, request_id, scheduled_time):
    return Publora().request(
        "POST",
        "create-post",
        json={
            "content": content,
            "platforms": platform_ids,
            "scheduledTime": scheduled_time,
        },
        idempotency_key=f"django-{request_id}",
    )

def enqueue_scheduled_post(content, platform_ids, request_id):
    scheduled_time = (timezone.now() + timedelta(minutes=5)).isoformat()
    schedule_post.delay(content, platform_ids, request_id, scheduled_time)
```

Compute `scheduled_time` before enqueueing it. Celery retries must reuse both the same timestamp and the same idempotency key; recomputing the timestamp changes the request body and returns `422 IDEMPOTENCY_KEY_CONFLICT`.

## Django webhook view

Read `request.body` unchanged and verify it before decoding JSON. The complete verification contract is in [Webhooks](../../endpoints/webhooks.md).

```python
# social/views.py
import hashlib
import hmac
import json
import os
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def publora_webhook(request):
    raw = request.body
    expected = hmac.new(
        os.environ["PUBLORA_WEBHOOK_SECRET"].encode(), raw, hashlib.sha256
    ).hexdigest()
    received = request.headers.get("x-publora-signature", "")
    if not hmac.compare_digest(received, expected):
        return HttpResponse(status=401)
    envelope = json.loads(raw)
    # Enqueue durable application work here.
    return HttpResponse(status=200)
```

## Workflow links

- [Draft and one-shot schedule](../curl/all-endpoints.md#1-create-a-draft)
- [Upload and schedule](../curl/all-endpoints.md#3-upload-and-schedule)
- [Update](../curl/all-endpoints.md#4-update-a-post)
- [Webhook consumer](../curl/all-endpoints.md#5-verify-a-webhook)
- [Complete OpenAPI surface](https://docs.publora.com/openapi.yaml)
