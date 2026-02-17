# Python Quick Start

Get started with the Publora API using Python and the `requests` library.

## Installation

```bash
pip install requests
```

## Setup

```python
import requests
from datetime import datetime, timedelta

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
}
```

## List Connected Accounts

```python
def get_connections():
    response = requests.get(
        f'{BASE_URL}/platform-connections',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    data = response.json()
    print('Connected accounts:', data['connections'])
    return data['connections']

# Example output:
# [
#   {'id': 'twitter-123456', 'platform': 'twitter', 'name': '@myaccount'},
#   {'id': 'linkedin-ABC123', 'platform': 'linkedin', 'name': 'John Doe'}
# ]
```

## Create a Simple Post

```python
def create_post(content, platforms):
    response = requests.post(
        f'{BASE_URL}/create-post',
        headers=headers,
        json={
            'content': content,
            'platforms': platforms
        }
    )

    data = response.json()
    print('Post created:', data['postGroupId'])
    return data

# Usage
create_post(
    'Hello from Publora API!',
    ['twitter-123456', 'linkedin-ABC123']
)
```

## Schedule a Post for Later

```python
def schedule_post(content, platforms, scheduled_time):
    response = requests.post(
        f'{BASE_URL}/create-post',
        headers=headers,
        json={
            'content': content,
            'platforms': platforms,
            'scheduledTime': scheduled_time  # ISO 8601 UTC format
        }
    )

    return response.json()

# Schedule for tomorrow at 2 PM UTC
tomorrow = datetime.utcnow() + timedelta(days=1)
tomorrow = tomorrow.replace(hour=14, minute=0, second=0, microsecond=0)

schedule_post(
    'Scheduled post going live tomorrow!',
    ['twitter-123456'],
    tomorrow.isoformat() + 'Z'
)
```

## Check Post Status

```python
def get_post_status(post_group_id):
    response = requests.get(
        f'{BASE_URL}/get-post/{post_group_id}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    data = response.json()

    # Check status of each platform
    for post in data['posts']:
        print(f"{post['platform']}: {post['status']}")

    return data

# Statuses: 'scheduled', 'published', 'failed', 'draft'
```

## Update a Scheduled Post

```python
def update_post(post_group_id, new_time=None, new_status=None):
    payload = {}

    if new_time:
        payload['scheduledTime'] = new_time
    if new_status:
        payload['status'] = new_status

    response = requests.put(
        f'{BASE_URL}/update-post/{post_group_id}',
        headers=headers,
        json=payload
    )

    return response.json()

# Reschedule a post
update_post('pg_abc123', new_time='2026-03-15T10:00:00.000Z')

# Move to draft
update_post('pg_abc123', new_status='draft')
```

## Delete a Post

```python
def delete_post(post_group_id):
    response = requests.delete(
        f'{BASE_URL}/delete-post/{post_group_id}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    if response.ok:
        print('Post deleted successfully')

    return response.json()
```

## Complete Example

```python
import requests
from datetime import datetime, timedelta

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

def main():
    headers = {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY
    }

    # 1. Get connected accounts
    connections_response = requests.get(
        f'{BASE_URL}/platform-connections',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )
    connections = connections_response.json()['connections']

    if not connections:
        print('No accounts connected. Visit app.publora.com to connect.')
        return

    # 2. Get platform IDs
    platform_ids = [c['id'] for c in connections]
    print('Posting to:', platform_ids)

    # 3. Create a post
    post_response = requests.post(
        f'{BASE_URL}/create-post',
        headers=headers,
        json={
            'content': 'Testing the Publora API - works great!',
            'platforms': platform_ids
        }
    )
    result = post_response.json()
    print('Created post:', result['postGroupId'])

    # 4. Check status
    import time
    time.sleep(5)

    status_response = requests.get(
        f'{BASE_URL}/get-post/{result["postGroupId"]}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )
    status = status_response.json()

    for post in status['posts']:
        print(f"{post['platform']}: {post['status']}")

if __name__ == '__main__':
    main()
```

## Error Handling

```python
def create_post_safe(content, platforms):
    try:
        response = requests.post(
            f'{BASE_URL}/create-post',
            headers=headers,
            json={
                'content': content,
                'platforms': platforms
            },
            timeout=30
        )

        response.raise_for_status()
        return {'success': True, 'data': response.json()}

    except requests.exceptions.HTTPError as e:
        error_data = e.response.json() if e.response else {}
        return {
            'success': False,
            'status_code': e.response.status_code if e.response else None,
            'error': error_data.get('message', str(e))
        }
    except requests.exceptions.Timeout:
        return {'success': False, 'error': 'Request timed out'}
    except requests.exceptions.RequestException as e:
        return {'success': False, 'error': str(e)}

# Usage
result = create_post_safe('Hello world!', ['twitter-123456'])

if result['success']:
    print('Post ID:', result['data']['postGroupId'])
else:
    print('Error:', result['error'])
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
