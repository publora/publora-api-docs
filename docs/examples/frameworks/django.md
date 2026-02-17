# Django Integration

Build social media features into your Django app with Publora API.

## Installation

```bash
pip install requests django
```

## Settings Configuration

```python
# settings.py
PUBLORA_API_KEY = os.environ.get('PUBLORA_API_KEY')
PUBLORA_BASE_URL = 'https://api.publora.com/api/v1'
```

## Publora Service Class

```python
# social/services.py
import requests
from django.conf import settings


class PubloraError(Exception):
    def __init__(self, status_code, message, body=None):
        super().__init__(message)
        self.status_code = status_code
        self.body = body or {}


class PubloraService:
    BASE_URL = settings.PUBLORA_BASE_URL

    def __init__(self, api_key=None, user_id=None):
        self.api_key = api_key or settings.PUBLORA_API_KEY
        self.user_id = user_id

    def _get_headers(self):
        headers = {
            'Content-Type': 'application/json',
            'x-publora-key': self.api_key,
        }
        if self.user_id:
            headers['x-publora-user-id'] = self.user_id
        return headers

    def _request(self, method, endpoint, data=None):
        url = f'{self.BASE_URL}{endpoint}'
        response = requests.request(
            method,
            url,
            headers=self._get_headers(),
            json=data
        )

        try:
            body = response.json()
        except ValueError:
            body = {}

        if not response.ok:
            message = body.get('error') or body.get('message') or 'API error'
            raise PubloraError(response.status_code, message, body)

        return body

    def get_connections(self):
        result = self._request('GET', '/platform-connections')
        return result.get('connections', [])

    def create_post(self, content, platforms, scheduled_time=None, platform_settings=None):
        data = {
            'content': content,
            'platforms': platforms,
        }
        if scheduled_time:
            data['scheduledTime'] = scheduled_time
        if platform_settings:
            data['platformSettings'] = platform_settings

        return self._request('POST', '/create-post', data)

    def get_post(self, post_group_id):
        return self._request('GET', f'/get-post/{post_group_id}')

    def update_post(self, post_group_id, **updates):
        return self._request('PUT', f'/update-post/{post_group_id}', updates)

    def delete_post(self, post_group_id):
        return self._request('DELETE', f'/delete-post/{post_group_id}')

    def get_upload_url(self, file_name, content_type, post_group_id):
        return self._request('POST', '/get-upload-url', {
            'fileName': file_name,
            'contentType': content_type,
            'postGroupId': post_group_id,
        })

    def get_linkedin_stats(self, platform_id, posted_id):
        return self._request('POST', '/linkedin-post-statistics', {
            'platformId': platform_id,
            'postedId': posted_id,
            'queryTypes': 'ALL',
        })


# Singleton instance
publora = PubloraService()
```

## Django Models

```python
# social/models.py
from django.db import models
from django.contrib.auth.models import User


class SocialPost(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('scheduled', 'Scheduled'),
        ('published', 'Published'),
        ('failed', 'Failed'),
        ('partially_published', 'Partially Published'),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    post_group_id = models.CharField(max_length=50, unique=True)
    content = models.TextField()
    platforms = models.JSONField(default=list)
    scheduled_time = models.DateTimeField(null=True, blank=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return f'{self.post_group_id} - {self.status}'

    def refresh_status(self):
        from .services import publora
        try:
            data = publora.get_post(self.post_group_id)
            self.status = data.get('status', self.status)
            self.save(update_fields=['status', 'updated_at'])
        except Exception as e:
            print(f'Failed to refresh status: {e}')


class PlatformConnection(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    platform_id = models.CharField(max_length=100)
    platform = models.CharField(max_length=50)
    username = models.CharField(max_length=100)
    display_name = models.CharField(max_length=200, blank=True)
    last_synced = models.DateTimeField(auto_now=True)

    class Meta:
        unique_together = ['user', 'platform_id']

    def __str__(self):
        return f'{self.platform}: {self.username}'
```

## Views

```python
# social/views.py
from django.http import JsonResponse
from django.views import View
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator
from django.contrib.auth.mixins import LoginRequiredMixin
import json

from .services import publora, PubloraError
from .models import SocialPost, PlatformConnection


@method_decorator(csrf_exempt, name='dispatch')
class ConnectionsView(LoginRequiredMixin, View):
    def get(self, request):
        try:
            connections = publora.get_connections()

            # Sync to local database
            for conn in connections:
                PlatformConnection.objects.update_or_create(
                    user=request.user,
                    platform_id=conn['platformId'],
                    defaults={
                        'platform': conn['platform'],
                        'username': conn['username'],
                        'display_name': conn.get('displayName', ''),
                    }
                )

            return JsonResponse({'connections': connections})
        except PubloraError as e:
            return JsonResponse({'error': str(e)}, status=e.status_code)


@method_decorator(csrf_exempt, name='dispatch')
class CreatePostView(LoginRequiredMixin, View):
    def post(self, request):
        try:
            data = json.loads(request.body)
            content = data.get('content')
            platforms = data.get('platforms', [])
            scheduled_time = data.get('scheduledTime')

            if not content or not platforms:
                return JsonResponse(
                    {'error': 'Content and platforms are required'},
                    status=400
                )

            result = publora.create_post(
                content=content,
                platforms=platforms,
                scheduled_time=scheduled_time
            )

            # Save to local database
            SocialPost.objects.create(
                user=request.user,
                post_group_id=result['postGroupId'],
                content=content,
                platforms=platforms,
                scheduled_time=scheduled_time,
                status='scheduled' if scheduled_time else 'published'
            )

            return JsonResponse(result, status=201)
        except PubloraError as e:
            return JsonResponse({'error': str(e)}, status=e.status_code)
        except json.JSONDecodeError:
            return JsonResponse({'error': 'Invalid JSON'}, status=400)


@method_decorator(csrf_exempt, name='dispatch')
class PostDetailView(LoginRequiredMixin, View):
    def get(self, request, post_group_id):
        try:
            result = publora.get_post(post_group_id)
            return JsonResponse(result)
        except PubloraError as e:
            return JsonResponse({'error': str(e)}, status=e.status_code)

    def delete(self, request, post_group_id):
        try:
            result = publora.delete_post(post_group_id)

            # Update local database
            SocialPost.objects.filter(
                user=request.user,
                post_group_id=post_group_id
            ).delete()

            return JsonResponse(result)
        except PubloraError as e:
            return JsonResponse({'error': str(e)}, status=e.status_code)
```

## URL Configuration

```python
# social/urls.py
from django.urls import path
from . import views

app_name = 'social'

urlpatterns = [
    path('connections/', views.ConnectionsView.as_view(), name='connections'),
    path('posts/', views.CreatePostView.as_view(), name='create_post'),
    path('posts/<str:post_group_id>/', views.PostDetailView.as_view(), name='post_detail'),
]

# project/urls.py
from django.urls import path, include

urlpatterns = [
    path('api/social/', include('social.urls')),
]
```

## Django REST Framework Integration

```python
# social/serializers.py
from rest_framework import serializers
from .models import SocialPost, PlatformConnection


class PlatformConnectionSerializer(serializers.ModelSerializer):
    class Meta:
        model = PlatformConnection
        fields = ['platform_id', 'platform', 'username', 'display_name']


class SocialPostSerializer(serializers.ModelSerializer):
    class Meta:
        model = SocialPost
        fields = ['post_group_id', 'content', 'platforms', 'scheduled_time', 'status', 'created_at']
        read_only_fields = ['post_group_id', 'status', 'created_at']


class CreatePostSerializer(serializers.Serializer):
    content = serializers.CharField(max_length=10000)
    platforms = serializers.ListField(child=serializers.CharField())
    scheduled_time = serializers.DateTimeField(required=False, allow_null=True)
```

```python
# social/api_views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

from .models import SocialPost, PlatformConnection
from .serializers import SocialPostSerializer, PlatformConnectionSerializer, CreatePostSerializer
from .services import publora, PubloraError


class PlatformConnectionViewSet(viewsets.ReadOnlyModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = PlatformConnectionSerializer

    def get_queryset(self):
        return PlatformConnection.objects.filter(user=self.request.user)

    @action(detail=False, methods=['post'])
    def sync(self, request):
        try:
            connections = publora.get_connections()

            for conn in connections:
                PlatformConnection.objects.update_or_create(
                    user=request.user,
                    platform_id=conn['platformId'],
                    defaults={
                        'platform': conn['platform'],
                        'username': conn['username'],
                        'display_name': conn.get('displayName', ''),
                    }
                )

            return Response({'synced': len(connections)})
        except PubloraError as e:
            return Response({'error': str(e)}, status=e.status_code)


class SocialPostViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = SocialPostSerializer

    def get_queryset(self):
        return SocialPost.objects.filter(user=self.request.user)

    def create(self, request):
        serializer = CreatePostSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        try:
            result = publora.create_post(
                content=serializer.validated_data['content'],
                platforms=serializer.validated_data['platforms'],
                scheduled_time=serializer.validated_data.get('scheduled_time'),
            )

            post = SocialPost.objects.create(
                user=request.user,
                post_group_id=result['postGroupId'],
                content=serializer.validated_data['content'],
                platforms=serializer.validated_data['platforms'],
                scheduled_time=serializer.validated_data.get('scheduled_time'),
                status='scheduled'
            )

            return Response(
                SocialPostSerializer(post).data,
                status=status.HTTP_201_CREATED
            )
        except PubloraError as e:
            return Response({'error': str(e)}, status=e.status_code)

    @action(detail=True, methods=['post'])
    def refresh(self, request, pk=None):
        post = self.get_object()
        post.refresh_status()
        return Response(SocialPostSerializer(post).data)

    def destroy(self, request, pk=None):
        post = self.get_object()
        try:
            publora.delete_post(post.post_group_id)
            post.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)
        except PubloraError as e:
            return Response({'error': str(e)}, status=e.status_code)
```

## Celery Tasks for Background Processing

```python
# social/tasks.py
from celery import shared_task
from django.utils import timezone
from .models import SocialPost
from .services import publora, PubloraError


@shared_task
def refresh_post_statuses():
    """Refresh status of all pending posts."""
    pending_posts = SocialPost.objects.filter(
        status__in=['scheduled', 'processing']
    )

    for post in pending_posts:
        try:
            data = publora.get_post(post.post_group_id)
            post.status = data.get('status', post.status)
            post.save(update_fields=['status', 'updated_at'])
        except PubloraError as e:
            print(f'Failed to refresh {post.post_group_id}: {e}')


@shared_task
def schedule_post(post_id):
    """Schedule a post via Publora API."""
    try:
        post = SocialPost.objects.get(id=post_id)

        result = publora.create_post(
            content=post.content,
            platforms=post.platforms,
            scheduled_time=post.scheduled_time.isoformat() if post.scheduled_time else None
        )

        post.post_group_id = result['postGroupId']
        post.status = 'scheduled'
        post.save()

        return {'success': True, 'post_group_id': result['postGroupId']}
    except SocialPost.DoesNotExist:
        return {'success': False, 'error': 'Post not found'}
    except PubloraError as e:
        return {'success': False, 'error': str(e)}


@shared_task
def bulk_schedule_posts(post_ids):
    """Schedule multiple posts with rate limiting."""
    import time

    results = []
    for post_id in post_ids:
        result = schedule_post.delay(post_id)
        results.append({'post_id': post_id, 'task_id': result.id})
        time.sleep(0.2)  # Rate limiting

    return results
```

## Management Command

```python
# social/management/commands/sync_connections.py
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from social.services import publora
from social.models import PlatformConnection


class Command(BaseCommand):
    help = 'Sync platform connections from Publora'

    def add_arguments(self, parser):
        parser.add_argument('--user', type=str, help='Username to sync for')

    def handle(self, *args, **options):
        try:
            connections = publora.get_connections()

            self.stdout.write(f'Found {len(connections)} connections')

            for conn in connections:
                self.stdout.write(f"  - {conn['platform']}: {conn['username']}")

            self.stdout.write(self.style.SUCCESS('Sync complete'))
        except Exception as e:
            self.stdout.write(self.style.ERROR(f'Sync failed: {e}'))
```

## Template Example

```html
<!-- templates/social/post_form.html -->
{% extends 'base.html' %}

{% block content %}
<div class="container">
  <h1>Create Social Post</h1>

  <form id="post-form" method="post">
    {% csrf_token %}

    <div class="form-group">
      <label for="content">Content</label>
      <textarea id="content" name="content" class="form-control" rows="4" required></textarea>
      <small class="text-muted"><span id="char-count">0</span> characters</small>
    </div>

    <div class="form-group">
      <label>Platforms</label>
      <div id="platforms">
        {% for conn in connections %}
        <div class="form-check">
          <input type="checkbox" class="form-check-input" name="platforms" value="{{ conn.platform_id }}" id="platform-{{ conn.platform_id }}">
          <label class="form-check-label" for="platform-{{ conn.platform_id }}">
            {{ conn.platform }}: {{ conn.username }}
          </label>
        </div>
        {% endfor %}
      </div>
    </div>

    <div class="form-group">
      <label for="scheduled_time">Schedule (optional)</label>
      <input type="datetime-local" id="scheduled_time" name="scheduled_time" class="form-control">
    </div>

    <button type="submit" class="btn btn-primary">Schedule Post</button>
  </form>

  <div id="result" class="mt-3" style="display: none;"></div>
</div>

<script>
document.getElementById('content').addEventListener('input', function() {
  document.getElementById('char-count').textContent = this.value.length;
});

document.getElementById('post-form').addEventListener('submit', async function(e) {
  e.preventDefault();

  const formData = new FormData(this);
  const platforms = formData.getAll('platforms');

  const response = await fetch('/api/social/posts/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRFToken': formData.get('csrfmiddlewaretoken'),
    },
    body: JSON.stringify({
      content: formData.get('content'),
      platforms: platforms,
      scheduledTime: formData.get('scheduled_time') || null,
    }),
  });

  const result = await response.json();
  const resultDiv = document.getElementById('result');
  resultDiv.style.display = 'block';

  if (response.ok) {
    resultDiv.className = 'alert alert-success';
    resultDiv.textContent = `Post created: ${result.post_group_id}`;
  } else {
    resultDiv.className = 'alert alert-danger';
    resultDiv.textContent = `Error: ${result.error}`;
  }
});
</script>
{% endblock %}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
