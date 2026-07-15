# Ruby Quick Start

Ruby syntax for the five [Core Workflows](../curl/all-endpoints.md). The canonical page owns the API semantics.

## Client

```ruby
require 'json'
require 'net/http'
require 'securerandom'
require 'time'
require 'uri'

BASE_URL = 'https://api.publora.com/api/v1'

def publora(method, path, body: nil, idempotency_key: nil)
  uri = URI("#{BASE_URL}/#{path}")
  request = Net::HTTP.const_get(method.capitalize).new(uri)
  request['x-publora-key'] = ENV.fetch('PUBLORA_API_KEY')
  request['Content-Type'] = 'application/json'
  request['Idempotency-Key'] = idempotency_key if idempotency_key
  request.body = JSON.generate(body) if body
  response = Net::HTTP.start(uri.host, uri.port, use_ssl: true) { |http| http.request(request) }
  raise "Publora returned #{response.code}" unless response.is_a?(Net::HTTPSuccess)
  JSON.parse(response.body)
end

platform_id = ENV.fetch('PUBLORA_PLATFORM_ID')
```

## 1. Draft

```ruby
draft = publora('post', 'create-post', body: {
  content: 'Draft from Ruby', platforms: [platform_id]
})
```

No `scheduledTime` means draft.

## 2. One-shot schedule

```ruby
scheduled = publora('post', 'create-post', idempotency_key: SecureRandom.uuid, body: {
  content: 'Scheduled from Ruby',
  platforms: [platform_id],
  scheduledTime: (Time.now.utc + 300).iso8601
})
```

## 3. Upload and schedule

Follow the [canonical presigned sequence](../curl/all-endpoints.md#3-upload-and-schedule). PUT bytes to the returned storage URL without the Publora API header, then:

```ruby
upload_uri = URI(upload_url)
upload_request = Net::HTTP::Put.new(upload_uri)
upload_request['Content-Type'] = 'image/jpeg'
upload_request.body = File.binread('photo.jpg')
upload_response = Net::HTTP.start(upload_uri.host, upload_uri.port, use_ssl: true) do |http|
  http.request(upload_request)
end
raise 'Storage upload failed' unless upload_response.is_a?(Net::HTTPSuccess)

publora('post', "complete-media/#{media_file_id}")
publora('put', "update-post/#{post_group_id}",
  idempotency_key: "schedule-#{post_group_id}", body: {
    status: 'scheduled', scheduledTime: (Time.now.utc + 300).iso8601
  })
```

## 4. Update

```ruby
publora('put', "update-post/#{post_group_id}",
  idempotency_key: "reschedule-#{post_group_id}", body: {
    status: 'scheduled', scheduledTime: (Time.now.utc + 600).iso8601
  })
```

## 5. Webhook consumer

```ruby
require 'openssl'

post '/publora-webhook' do
  raw = request.body.read
  expected = OpenSSL::HMAC.hexdigest('SHA256', ENV.fetch('PUBLORA_WEBHOOK_SECRET'), raw)
  halt 401 unless Rack::Utils.secure_compare(expected, request.env['HTTP_X_PUBLORA_SIGNATURE'].to_s)
  envelope = JSON.parse(raw)
  status 204
end
```

Rails/Faraday/HTTParty may wrap the same calls; they do not change the wire contract. CSV import, batch operations, and LinkedIn analytics remain discoverable through dedicated pages and the [OpenAPI document](https://docs.publora.com/openapi.yaml). See [Webhooks](../../endpoints/webhooks.md).
