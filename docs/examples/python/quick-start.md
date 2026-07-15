# Python Quick Start

Python syntax for the five [Core Workflows](../curl/all-endpoints.md). The canonical page owns the API semantics.

## Setup

```bash
pip install requests flask
```

```python
import os
import requests

BASE_URL = "https://api.publora.com/api/v1"
API_KEY = os.environ["PUBLORA_API_KEY"]
PLATFORM_ID = os.environ["PUBLORA_PLATFORM_ID"]

def publora(method, path, *, body=None, idempotency_key=None):
    headers = {"x-publora-key": API_KEY}
    if idempotency_key:
        headers["Idempotency-Key"] = idempotency_key
    response = requests.request(
        method, f"{BASE_URL}/{path}", headers=headers, json=body, timeout=30
    )
    response.raise_for_status()
    return response.json()
```

## 1. Draft

```python
draft = publora("POST", "create-post", body={
    "content": "Draft from Python",
    "platforms": [PLATFORM_ID],
})
```

Omitting `scheduledTime` intentionally creates a draft.

## 2. One-shot schedule

```python
from datetime import datetime, timedelta, timezone
from uuid import uuid4

scheduled = publora("POST", "create-post", idempotency_key=str(uuid4()), body={
    "content": "Scheduled from Python",
    "platforms": [PLATFORM_ID],
    "scheduledTime": (datetime.now(timezone.utc) + timedelta(minutes=5)).isoformat(),
})
```

## 3. Upload and schedule

Use the [canonical presigned sequence](../curl/all-endpoints.md#3-upload-and-schedule). After creating the draft and requesting an upload URL:

```python
with open("photo.jpg", "rb") as image:
    uploaded = requests.put(
        upload_url,
        headers={"Content-Type": "image/jpeg"},
        data=image,
        timeout=120,
    )
    uploaded.raise_for_status()

publora("POST", f"complete-media/{media_file_id}")
publora("PUT", f"update-post/{post_group_id}",
    idempotency_key=f"schedule-{post_group_id}", body={
        "status": "scheduled",
        "scheduledTime": (datetime.now(timezone.utc) + timedelta(minutes=5)).isoformat(),
    })
```

## 4. Update

```python
publora("PUT", f"update-post/{post_group_id}",
    idempotency_key=f"reschedule-{post_group_id}", body={
        "status": "scheduled",
        "scheduledTime": (datetime.now(timezone.utc) + timedelta(minutes=10)).isoformat(),
    })
```

## 5. Webhook consumer

```python
import hashlib
import hmac
import json
from flask import Flask, abort, request

app = Flask(__name__)

@app.post("/publora-webhook")
def webhook():
    raw = request.get_data()  # capture before request.get_json()
    expected = hmac.new(
        os.environ["PUBLORA_WEBHOOK_SECRET"].encode(), raw, hashlib.sha256
    ).hexdigest()
    if not hmac.compare_digest(request.headers.get("x-publora-signature", ""), expected):
        abort(401)
    envelope = json.loads(raw)
    return "", 204
```

See [Webhooks](../../endpoints/webhooks.md) and the [complete OpenAPI surface](https://docs.publora.com/openapi.yaml).
