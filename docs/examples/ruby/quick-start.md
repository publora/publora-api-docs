# Ruby Quick Start

Integrate Publora API with Ruby using Net::HTTP or popular gems.

## Using Net::HTTP (No Dependencies)

```ruby
# publora_client.rb
require 'net/http'
require 'json'
require 'uri'

class PubloraError < StandardError
  attr_reader :status_code, :body

  def initialize(status_code, message, body = {})
    super(message)
    @status_code = status_code
    @body = body
  end
end

class PubloraClient
  BASE_URL = 'https://api.publora.com/api/v1'.freeze

  def initialize(api_key, user_id: nil)
    @api_key = api_key
    @user_id = user_id
  end

  def get_connections
    response = request(:get, '/platform-connections')
    response['connections'] || []
  end

  def create_post(content:, platforms:, scheduled_time: nil, platform_settings: nil)
    body = {
      content: content,
      platforms: platforms
    }
    body[:scheduledTime] = scheduled_time if scheduled_time
    body[:platformSettings] = platform_settings if platform_settings

    request(:post, '/create-post', body)
  end

  def get_post(post_group_id)
    request(:get, "/get-post/#{post_group_id}")
  end

  def update_post(post_group_id, **updates)
    request(:put, "/update-post/#{post_group_id}", updates)
  end

  def delete_post(post_group_id)
    request(:delete, "/delete-post/#{post_group_id}")
  end

  def get_upload_url(file_name, content_type, post_group_id)
    request(:post, '/get-upload-url', {
      fileName: file_name,
      contentType: content_type,
      postGroupId: post_group_id
    })
  end

  def linkedin_post_stats(platform_id, posted_id)
    request(:post, '/linkedin-post-statistics', {
      platformId: platform_id,
      postedId: posted_id,
      queryTypes: 'ALL'
    })
  end

  private

  def request(method, endpoint, body = nil)
    uri = URI("#{BASE_URL}#{endpoint}")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true

    request = case method
              when :get then Net::HTTP::Get.new(uri)
              when :post then Net::HTTP::Post.new(uri)
              when :put then Net::HTTP::Put.new(uri)
              when :delete then Net::HTTP::Delete.new(uri)
              end

    request['Content-Type'] = 'application/json'
    request['x-publora-key'] = @api_key
    request['x-publora-user-id'] = @user_id if @user_id
    request.body = body.to_json if body

    response = http.request(request)
    parsed_body = JSON.parse(response.body) rescue {}

    unless response.is_a?(Net::HTTPSuccess)
      message = parsed_body['error'] || parsed_body['message'] || 'Unknown API error'
      raise PubloraError.new(response.code.to_i, message, parsed_body)
    end

    parsed_body
  end
end
```

## Usage Examples

### Initialize Client

```ruby
require_relative 'publora_client'

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])
```

### List Platform Connections

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

begin
  connections = client.get_connections

  puts "Found #{connections.length} connected accounts:"
  connections.each do |conn|
    puts "  - #{conn['platform']}: #{conn['username']} (#{conn['platformId']})"
  end
rescue PubloraError => e
  puts "Error: #{e.message}"
end
```

### Create a Simple Post

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

begin
  response = client.create_post(
    content: 'Hello from Ruby! Posting with Publora API.',
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF']
  )

  puts "Post created: #{response['postGroupId']}"
  response['posts'].each do |post|
    puts "  - #{post['platform']}: #{post['status']}"
  end
rescue PubloraError => e
  puts "Failed to create post: #{e.message}"
end
```

### Schedule a Post

```ruby
require 'time'

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

# Schedule for tomorrow at 10 AM UTC
scheduled_time = (Time.now.utc + 86400).strftime('%Y-%m-%dT10:00:00.000Z')

begin
  response = client.create_post(
    content: 'This post will go live tomorrow at 10 AM UTC!',
    platforms: ['twitter-123456789'],
    scheduled_time: scheduled_time
  )

  puts "Post scheduled: #{response['postGroupId']}"
  puts "Will publish at: #{scheduled_time}"
rescue PubloraError => e
  puts "Failed to schedule: #{e.message}"
end
```

### Schedule Multiple Posts (Week of Content)

```ruby
require 'time'

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

posts = [
  { content: 'Monday motivation: Start your week with purpose!', day_offset: 0 },
  { content: 'Tech tip Tuesday: Always version your APIs.', day_offset: 1 },
  { content: 'Wednesday wisdom: Ship fast, iterate faster.', day_offset: 2 },
  { content: 'Throwback Thursday: How we grew to 10K users.', day_offset: 3 },
  { content: 'Feature Friday: Check out our new dashboard!', day_offset: 4 }
]

base_time = Time.now.utc.to_date.to_time + (9 * 3600) # 9 AM UTC

posts.each_with_index do |post, index|
  scheduled_time = (base_time + (post[:day_offset] * 86400)).iso8601

  begin
    response = client.create_post(
      content: post[:content],
      platforms: ['twitter-123456789', 'linkedin-ABC123DEF'],
      scheduled_time: scheduled_time
    )

    puts "Scheduled: #{post[:content][0..29]}... -> #{response['postGroupId']}"
    sleep(0.2) # Rate limiting
  rescue PubloraError => e
    puts "Failed: #{e.message}"
  end
end
```

### Create a Post (Media Auto-Attaches via postGroupId)

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

begin
  response = client.create_post(
    content: 'Check out this amazing screenshot!',
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF']
  )

  puts "Post created: #{response['postGroupId']}"
rescue PubloraError => e
  puts "Failed: #{e.message}"
end
```

### Post with Platform-Specific Settings

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

# Instagram Reel
begin
  response = client.create_post(
    content: 'Behind the scenes! #buildinpublic',
    platforms: ['instagram-789012345'],
    platform_settings: {
      instagram: {
        videoType: 'REELS'
      }
    }
  )

  puts "Instagram Reel scheduled: #{response['postGroupId']}"
rescue PubloraError => e
  puts "Failed: #{e.message}"
end

# Telegram with Markdown
begin
  response = client.create_post(
    content: '*Bold* and _italic_ text with [link](https://example.com)',
    platforms: ['telegram-1001234567890'],
    platform_settings: {
      telegram: {
        parseMode: 'MarkdownV2',
        disableWebPagePreview: false
      }
    }
  )

  puts "Telegram message scheduled: #{response['postGroupId']}"
rescue PubloraError => e
  puts "Failed: #{e.message}"
end
```

### Upload Media and Post

```ruby
require 'net/http'

def get_mime_type(filename)
  ext = File.extname(filename).downcase
  {
    '.jpg' => 'image/jpeg',
    '.jpeg' => 'image/jpeg',
    '.png' => 'image/png',
    '.gif' => 'image/gif',
    '.mp4' => 'video/mp4',
    '.webp' => 'image/webp'
  }[ext] || 'application/octet-stream'
end

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])
file_path = './screenshot.png'

begin
  # Step 1: Create post first to get postGroupId
  post_response = client.create_post(
    content: 'Check out this screenshot!',
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF']
  )

  post_group_id = post_response['postGroupId']
  puts "Post created: #{post_group_id}"

  # Step 2: Get upload URL (media auto-attaches via postGroupId)
  file_name = File.basename(file_path)
  content_type = get_mime_type(file_name)

  upload_result = client.get_upload_url(file_name, content_type, post_group_id)

  # Step 3: Upload file to S3
  uri = URI(upload_result['uploadUrl'])
  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true

  request = Net::HTTP::Put.new(uri)
  request['Content-Type'] = content_type
  request.body = File.read(file_path, mode: 'rb')

  response = http.request(request)
  raise "Upload failed: #{response.code}" unless response.is_a?(Net::HTTPSuccess)

  puts "Media uploaded: #{upload_result['mediaId']}"
  puts "File URL: #{upload_result['fileUrl']}"
  puts "Post with uploaded media created: #{post_group_id}"
rescue => e
  puts "Error: #{e.message}"
end
```

### Error Handling

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

begin
  response = client.create_post(
    content: 'Test post',
    platforms: ['twitter-123456789']
  )

  puts "Success: #{response['postGroupId']}"
rescue PubloraError => e
  case e.status_code
  when 401
    puts 'Authentication failed. Check your API key.'
  when 403
    puts "Access denied: #{e.message}"
  when 400
    puts "Bad request: #{e.message}"
  when 404
    puts "Not found: #{e.message}"
  when 429
    puts 'Rate limited. Retry after delay.'
  when 500
    puts 'Server error. Retry later.'
  else
    puts "API error (#{e.status_code}): #{e.message}"
  end
rescue StandardError => e
  puts "Network or unexpected error: #{e.message}"
end
```

### Monitor Post Status

```ruby
def wait_for_publish(client, post_group_id, max_attempts: 30, interval: 10)
  max_attempts.times do |attempt|
    post_group = client.get_post(post_group_id)
    status = post_group['status']

    puts "[#{attempt + 1}/#{max_attempts}] Status: #{status}"

    case status
    when 'published'
      puts 'All platforms published successfully!'
      return post_group
    when 'partially_published'
      puts 'Partial success:'
      post_group['posts'].each do |post|
        if post['status'] == 'failed'
          puts "  - #{post['platform']}: FAILED - #{post['error']}"
        else
          puts "  - #{post['platform']}: #{post['status']}"
        end
      end
      return post_group
    when 'failed'
      puts 'All platforms failed:'
      post_group['posts'].each do |post|
        puts "  - #{post['platform']}: #{post['error']}"
      end
      return post_group
    end

    sleep(interval)
  end

  raise "Timeout waiting for post #{post_group_id} to publish"
end

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

begin
  response = client.create_post(
    content: 'Publishing now!',
    platforms: ['twitter-123456789']
  )

  post_group = wait_for_publish(client, response['postGroupId'])
  puts "Final status: #{post_group['status']}"
rescue => e
  puts "Error: #{e.message}"
end
```

### Import from CSV

```ruby
require 'csv'

client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

results = []

CSV.foreach('posts.csv', headers: true) do |row|
  platforms = row['platforms'].split(';').map(&:strip)

  post_data = {
    content: row['content'],
    platforms: platforms,
    scheduled_time: row['scheduled_time']
  }

  begin
    response = client.create_post(**post_data)
    results << { content: row['content'][0..29] + '...', success: true, post_group_id: response['postGroupId'] }
    puts "✓ #{row['content'][0..39]}..."
  rescue PubloraError => e
    results << { content: row['content'][0..29] + '...', success: false, error: e.message }
    puts "✗ #{row['content'][0..39]}... - #{e.message}"
  end

  sleep(0.2) # Rate limiting
end

successful = results.count { |r| r[:success] }
puts "\nImported #{successful}/#{results.length} posts"
```

### LinkedIn Analytics

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

platform_id = 'linkedin-ABC123DEF'
posted_ids = [
  'urn:li:share:7123456789012345678',
  'urn:li:share:7234567890123456789'
]

puts '=== LinkedIn Analytics Report ==='
puts

total_impressions = 0
total_engagement = 0

posted_ids.each do |posted_id|
  begin
    result = client.linkedin_post_stats(platform_id, posted_id)

    metrics = result['metrics']
    engagement = metrics['REACTION'] + metrics['COMMENT'] + metrics['RESHARE']
    total_impressions += metrics['IMPRESSION']
    total_engagement += engagement

    puts "Post: #{posted_id}"
    puts "  Impressions: #{metrics['IMPRESSION'].to_s.reverse.gsub(/(\d{3})(?=\d)/, '\\1,').reverse}"
    puts "  Members Reached: #{metrics['MEMBERS_REACHED'].to_s.reverse.gsub(/(\d{3})(?=\d)/, '\\1,').reverse}"
    puts "  Engagement: #{engagement} (#{metrics['REACTION']} reactions, #{metrics['COMMENT']} comments, #{metrics['RESHARE']} reshares)"
    puts
  rescue PubloraError => e
    puts "Post: #{posted_id} - Error: #{e.message}"
    puts
  end
end

puts '=== TOTALS ==='
puts "Total Impressions: #{total_impressions.to_s.reverse.gsub(/(\d{3})(?=\d)/, '\\1,').reverse}"
puts "Total Engagement: #{total_engagement}"
if total_impressions > 0
  puts "Avg Engagement Rate: #{'%.2f' % ((total_engagement.to_f / total_impressions) * 100)}%"
end
```

### Batch Delete Posts

```ruby
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

post_group_ids = ['pg_abc123', 'pg_def456', 'pg_ghi789']

results = []

post_group_ids.each do |id|
  begin
    client.delete_post(id)
    results << { id: id, success: true }
    puts "✓ Deleted #{id}"
  rescue PubloraError => e
    results << { id: id, success: false, error: e.message }
    puts "✗ Failed to delete #{id}: #{e.message}"
  end

  sleep(0.1) # Rate limiting
end

successful = results.count { |r| r[:success] }
puts "\nDeleted #{successful}/#{post_group_ids.length} posts"
```

## Using HTTParty Gem

```ruby
# Gemfile
# gem 'httparty'

require 'httparty'

class PubloraAPI
  include HTTParty
  base_uri 'https://api.publora.com/api/v1'

  def initialize(api_key)
    @api_key = api_key
  end

  def get_connections
    response = self.class.get('/platform-connections', headers: auth_headers)
    handle_response(response)['connections']
  end

  def create_post(content:, platforms:, scheduled_time: nil)
    body = { content: content, platforms: platforms }
    body[:scheduledTime] = scheduled_time if scheduled_time

    response = self.class.post('/create-post',
      headers: auth_headers.merge('Content-Type' => 'application/json'),
      body: body.to_json
    )
    handle_response(response)
  end

  private

  def auth_headers
    { 'x-publora-key' => @api_key }
  end

  def handle_response(response)
    raise "API Error (#{response.code}): #{response['error'] || response['message']}" unless response.success?
    response.parsed_response
  end
end

# Usage
api = PubloraAPI.new(ENV['PUBLORA_API_KEY'])

connections = api.get_connections
puts "Connections: #{connections.length}"

post = api.create_post(
  content: 'Hello from HTTParty!',
  platforms: ['twitter-123456789']
)
puts "Created: #{post['postGroupId']}"
```

## Using Faraday Gem

```ruby
# Gemfile
# gem 'faraday'

require 'faraday'
require 'json'

class PubloraClient
  BASE_URL = 'https://api.publora.com/api/v1'

  def initialize(api_key)
    @conn = Faraday.new(url: BASE_URL) do |f|
      f.request :json
      f.response :json
      f.adapter Faraday.default_adapter
      f.headers['x-publora-key'] = api_key
    end
  end

  def get_connections
    response = @conn.get('platform-connections')
    handle_response(response)['connections']
  end

  def create_post(content:, platforms:, scheduled_time: nil)
    body = { content: content, platforms: platforms }
    body[:scheduledTime] = scheduled_time if scheduled_time

    response = @conn.post('create-post', body)
    handle_response(response)
  end

  def get_post(post_group_id)
    response = @conn.get("get-post/#{post_group_id}")
    handle_response(response)
  end

  private

  def handle_response(response)
    unless response.success?
      error = response.body['error'] || response.body['message'] || 'Unknown error'
      raise "API Error (#{response.status}): #{error}"
    end
    response.body
  end
end

# Usage
client = PubloraClient.new(ENV['PUBLORA_API_KEY'])

connections = client.get_connections
puts "Found #{connections.length} connections"

post = client.create_post(
  content: 'Hello from Faraday!',
  platforms: ['twitter-123456789', 'linkedin-ABC123DEF']
)
puts "Created post: #{post['postGroupId']}"
```

## Rails Integration

```ruby
# config/initializers/publora.rb
require_relative '../../lib/publora_client'

PUBLORA_CLIENT = PubloraClient.new(Rails.application.credentials.publora_api_key)

# app/services/social_posting_service.rb
class SocialPostingService
  def self.schedule_post(content:, platforms:, scheduled_time:)
    PUBLORA_CLIENT.create_post(
      content: content,
      platforms: platforms,
      scheduled_time: scheduled_time.iso8601
    )
  end

  def self.get_connections
    PUBLORA_CLIENT.get_connections
  end
end

# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def create
    result = SocialPostingService.schedule_post(
      content: params[:content],
      platforms: params[:platforms],
      scheduled_time: Time.parse(params[:scheduled_time])
    )

    render json: { success: true, post_group_id: result['postGroupId'] }
  rescue PubloraError => e
    render json: { success: false, error: e.message }, status: :unprocessable_entity
  end
end
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
