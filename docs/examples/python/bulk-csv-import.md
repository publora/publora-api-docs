# Python: Bulk Import from CSV

Schedule multiple posts from a CSV file using Python.

## CSV Format

Create a CSV file (`posts.csv`) with your content:

```csv
content,platforms,scheduled_time,media_url
"Monday motivation: Start strong!",twitter-123456;linkedin-ABC123,2026-03-01T09:00:00Z,
"Check out our new feature!",twitter-123456,2026-03-01T14:00:00Z,https://example.com/feature.png
"Weekly roundup thread",twitter-123456,2026-03-02T10:00:00Z,
"LinkedIn deep dive post",linkedin-ABC123,2026-03-02T12:00:00Z,https://example.com/chart.png
```

## Basic CSV Import

```python
import csv
import requests
from datetime import datetime

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
}

def import_from_csv(csv_file):
    results = []

    with open(csv_file, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)

        for row in reader:
            # Parse platforms (semicolon-separated)
            platforms = row['platforms'].split(';')

            # Build post payload
            payload = {
                'content': row['content'],
                'platforms': platforms,
                'scheduledTime': row['scheduled_time']
            }

            # Create the post
            response = requests.post(
                f'{BASE_URL}/create-post',
                headers=headers,
                json=payload
            )

            result = {
                'content': row['content'][:50] + '...',
                'scheduled': row['scheduled_time'],
                'success': response.ok
            }

            if response.ok:
                result['postGroupId'] = response.json()['postGroupId']
            else:
                result['error'] = response.json().get('message', 'Unknown error')

            results.append(result)
            print(f"{'✓' if response.ok else '✗'} {result['content']}")

    # Summary
    successful = sum(1 for r in results if r['success'])
    print(f"\nImported {successful}/{len(results)} posts")

    return results

# Usage
results = import_from_csv('posts.csv')
```

## Advanced CSV Import with Validation

```python
import csv
import requests
from datetime import datetime, timezone
import time

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
}

def validate_row(row, row_num):
    """Validate a CSV row before processing."""
    errors = []

    if not row.get('content'):
        errors.append(f"Row {row_num}: Missing content")

    if not row.get('platforms'):
        errors.append(f"Row {row_num}: Missing platforms")

    if row.get('scheduled_time'):
        try:
            dt = datetime.fromisoformat(row['scheduled_time'].replace('Z', '+00:00'))
            if dt < datetime.now(timezone.utc):
                errors.append(f"Row {row_num}: Scheduled time is in the past")
        except ValueError:
            errors.append(f"Row {row_num}: Invalid date format (use ISO 8601)")

    return errors

def import_csv_advanced(csv_file, dry_run=False, delay_ms=500):
    """
    Import posts from CSV with validation and rate limiting.

    Args:
        csv_file: Path to CSV file
        dry_run: If True, validate only without posting
        delay_ms: Delay between API calls in milliseconds
    """
    all_errors = []
    results = []

    # First pass: validate all rows
    with open(csv_file, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        rows = list(reader)

        print(f"Validating {len(rows)} rows...")

        for i, row in enumerate(rows, 1):
            errors = validate_row(row, i)
            all_errors.extend(errors)

    if all_errors:
        print("\nValidation errors:")
        for error in all_errors:
            print(f"  - {error}")
        return {'success': False, 'errors': all_errors}

    print("Validation passed!")

    if dry_run:
        print("\nDry run mode - no posts created")
        return {'success': True, 'dry_run': True, 'row_count': len(rows)}

    # Second pass: create posts
    print(f"\nCreating {len(rows)} posts...")

    for i, row in enumerate(rows, 1):
        platforms = row['platforms'].split(';')

        payload = {
            'content': row['content'],
            'platforms': platforms
        }

        if row.get('scheduled_time'):
            payload['scheduledTime'] = row['scheduled_time']

        response = requests.post(
            f'{BASE_URL}/create-post',
            headers=headers,
            json=payload
        )

        result = {
            'row': i,
            'success': response.ok,
            'content_preview': row['content'][:40]
        }

        if response.ok:
            result['postGroupId'] = response.json()['postGroupId']
            print(f"  [{i}/{len(rows)}] ✓ Created: {result['postGroupId']}")
        else:
            result['error'] = response.json().get('message', 'Unknown error')
            print(f"  [{i}/{len(rows)}] ✗ Failed: {result['error']}")

        results.append(result)

        # Rate limiting
        if i < len(rows):
            time.sleep(delay_ms / 1000)

    # Summary
    successful = sum(1 for r in results if r['success'])
    failed = len(results) - successful

    print(f"\n{'='*40}")
    print(f"Import complete: {successful} succeeded, {failed} failed")

    return {
        'success': failed == 0,
        'total': len(results),
        'successful': successful,
        'failed': failed,
        'results': results
    }

# Usage

# Validate only (dry run)
import_csv_advanced('posts.csv', dry_run=True)

# Actually import
import_csv_advanced('posts.csv', delay_ms=1000)
```

## Generate CSV Template

```python
def generate_csv_template(output_file='posts_template.csv'):
    """Generate a sample CSV template."""

    sample_rows = [
        {
            'content': 'Monday motivation: Start your week strong!',
            'platforms': 'twitter-123456;linkedin-ABC123',
            'scheduled_time': '2026-03-01T09:00:00Z',
            'media_url': ''
        },
        {
            'content': 'Check out our latest blog post!',
            'platforms': 'twitter-123456',
            'scheduled_time': '2026-03-01T14:00:00Z',
            'media_url': 'https://example.com/blog-image.png'
        },
        {
            'content': 'Behind the scenes at our office',
            'platforms': 'instagram-789012',
            'scheduled_time': '2026-03-02T11:00:00Z',
            'media_url': 'https://example.com/office-photo.jpg'
        }
    ]

    with open(output_file, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=['content', 'platforms', 'scheduled_time', 'media_url'])
        writer.writeheader()
        writer.writerows(sample_rows)

    print(f"Template created: {output_file}")

generate_csv_template()
```

## Export Scheduled Posts to CSV

```python
def export_scheduled_to_csv(output_file='scheduled_posts.csv'):
    """Export all scheduled posts to CSV for backup/review."""

    # Get all scheduled posts (you'd need to implement pagination for large sets)
    response = requests.get(
        f'{BASE_URL}/platform-connections',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    # This is a simplified example - actual implementation would
    # fetch posts differently based on your needs

    posts = []  # Fetch your scheduled posts here

    with open(output_file, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=[
            'postGroupId', 'content', 'platforms', 'scheduled_time', 'status'
        ])
        writer.writeheader()

        for post in posts:
            writer.writerow({
                'postGroupId': post['postGroupId'],
                'content': post['content'],
                'platforms': ';'.join(post['platforms']),
                'scheduled_time': post['scheduledTime'],
                'status': post['status']
            })

    print(f"Exported {len(posts)} posts to {output_file}")
```

## Using pandas for Larger Datasets

```python
import pandas as pd
import requests
from datetime import datetime
import time

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
}

def import_with_pandas(csv_file):
    """Import posts using pandas for better data handling."""

    df = pd.read_csv(csv_file)

    # Clean up data
    df['content'] = df['content'].str.strip()
    df['platforms'] = df['platforms'].str.split(';')
    df['scheduled_time'] = pd.to_datetime(df['scheduled_time'])

    # Filter out past dates
    now = datetime.now()
    df = df[df['scheduled_time'] > now]

    print(f"Processing {len(df)} future posts...")

    results = []

    for idx, row in df.iterrows():
        payload = {
            'content': row['content'],
            'platforms': row['platforms'],
            'scheduledTime': row['scheduled_time'].isoformat() + 'Z'
        }

        response = requests.post(
            f'{BASE_URL}/create-post',
            headers=headers,
            json=payload
        )

        results.append({
            'index': idx,
            'success': response.ok,
            'postGroupId': response.json().get('postGroupId') if response.ok else None
        })

        time.sleep(0.5)  # Rate limiting

    # Add results back to dataframe
    results_df = pd.DataFrame(results)
    df = df.reset_index(drop=True)
    df['import_success'] = results_df['success']
    df['postGroupId'] = results_df['postGroupId']

    # Save results
    df.to_csv('import_results.csv', index=False)

    success_count = df['import_success'].sum()
    print(f"Imported {success_count}/{len(df)} posts")

    return df

# Usage
df = import_with_pandas('posts.csv')
print(df[['content', 'import_success', 'postGroupId']])
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
