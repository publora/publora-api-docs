# Go Quick Start

Integrate Publora API with Go using the standard library and best practices.

## Installation

No external dependencies required. Uses Go's standard `net/http` package.

```bash
go mod init your-project
```

## Basic Client

```go
// publora/client.go
package publora

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"time"
)

const BaseURL = "https://api.publora.com/api/v1"

type Client struct {
	apiKey     string
	userId     string
	httpClient *http.Client
}

func NewClient(apiKey string) *Client {
	return &Client{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 30 * time.Second,
		},
	}
}

func NewWorkspaceClient(apiKey, userId string) *Client {
	return &Client{
		apiKey: apiKey,
		userId: userId,
		httpClient: &http.Client{
			Timeout: 30 * time.Second,
		},
	}
}

func (c *Client) request(method, endpoint string, body interface{}) ([]byte, error) {
	var reqBody io.Reader
	if body != nil {
		jsonData, err := json.Marshal(body)
		if err != nil {
			return nil, fmt.Errorf("failed to marshal request: %w", err)
		}
		reqBody = bytes.NewBuffer(jsonData)
	}

	req, err := http.NewRequest(method, BaseURL+endpoint, reqBody)
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %w", err)
	}

	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("x-publora-key", c.apiKey)
	if c.userId != "" {
		req.Header.Set("x-publora-user-id", c.userId)
	}

	resp, err := c.httpClient.Do(req)
	if err != nil {
		return nil, fmt.Errorf("request failed: %w", err)
	}
	defer resp.Body.Close()

	respBody, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("failed to read response: %w", err)
	}

	if resp.StatusCode >= 400 {
		var apiErr APIError
		json.Unmarshal(respBody, &apiErr)
		return nil, &apiErr
	}

	return respBody, nil
}
```

## Types

```go
// publora/types.go
package publora

import "fmt"

type APIError struct {
	StatusCode int    `json:"-"`
	Error      string `json:"error,omitempty"`
	Message    string `json:"message,omitempty"`
}

func (e *APIError) Error() string {
	msg := e.Error
	if msg == "" {
		msg = e.Message
	}
	return fmt.Sprintf("API error (%d): %s", e.StatusCode, msg)
}

type PlatformConnection struct {
	PlatformID    string `json:"platformId"`
	Platform      string `json:"platform"`
	Username      string `json:"username"`
	DisplayName   string `json:"displayName"`
	ProfileImage  string `json:"profileImageUrl,omitempty"`
	ExpiresAt     string `json:"accessTokenExpiresAt,omitempty"`
}

type Post struct {
	ID           string `json:"_id"`
	Platform     string `json:"platform"`
	PlatformID   string `json:"platformId"`
	Content      string `json:"content"`
	Status       string `json:"status"`
	PostedID     string `json:"postedId,omitempty"`
	PublishedURL string `json:"publishedUrl,omitempty"`
	Error        string `json:"error,omitempty"`
}

type PostGroup struct {
	PostGroupID   string `json:"postGroupId"`
	Content       string `json:"content"`
	Status        string `json:"status"`
	ScheduledTime string `json:"scheduledTime,omitempty"`
	Posts         []Post `json:"posts"`
}

type CreatePostRequest struct {
	Content          string                  `json:"content"`
	Platforms        []string                `json:"platforms"`
	ScheduledTime    string                  `json:"scheduledTime,omitempty"`
	MediaURLs        []string                `json:"mediaUrls,omitempty"`
	MediaKeys        []string                `json:"mediaKeys,omitempty"`
	Status           string                  `json:"status,omitempty"`
	PlatformSettings *PlatformSettings       `json:"platformSettings,omitempty"`
}

type PlatformSettings struct {
	Instagram *InstagramSettings `json:"instagram,omitempty"`
	TikTok    *TikTokSettings    `json:"tiktok,omitempty"`
	Telegram  *TelegramSettings  `json:"telegram,omitempty"`
}

type InstagramSettings struct {
	VideoType string `json:"videoType,omitempty"`
}

type TikTokSettings struct {
	DisableDuet    bool `json:"disableDuet,omitempty"`
	DisableStitch  bool `json:"disableStitch,omitempty"`
	DisableComment bool `json:"disableComment,omitempty"`
}

type TelegramSettings struct {
	ParseMode             string `json:"parseMode,omitempty"`
	DisableWebPagePreview bool   `json:"disableWebPagePreview,omitempty"`
}

type CreatePostResponse struct {
	Success     bool   `json:"success"`
	PostGroupID string `json:"postGroupId"`
	Posts       []struct {
		Platform             string `json:"platform"`
		PlatformConnectionID string `json:"platformConnectionId"`
		Status               string `json:"status"`
	} `json:"posts"`
}

type UploadURLResponse struct {
	UploadURL string `json:"uploadUrl"`
	MediaKey  string `json:"mediaKey"`
}

type LinkedInStats struct {
	Impressions    int     `json:"impressions"`
	Clicks         int     `json:"clicks"`
	Likes          int     `json:"likes"`
	Comments       int     `json:"comments"`
	Shares         int     `json:"shares"`
	EngagementRate float64 `json:"engagementRate,omitempty"`
}
```

## API Methods

```go
// publora/api.go
package publora

import "encoding/json"

func (c *Client) GetConnections() ([]PlatformConnection, error) {
	data, err := c.request("GET", "/platform-connections", nil)
	if err != nil {
		return nil, err
	}

	var result struct {
		Connections []PlatformConnection `json:"connections"`
	}
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return result.Connections, nil
}

func (c *Client) CreatePost(req *CreatePostRequest) (*CreatePostResponse, error) {
	data, err := c.request("POST", "/create-post", req)
	if err != nil {
		return nil, err
	}

	var result CreatePostResponse
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return &result, nil
}

func (c *Client) GetPost(postGroupID string) (*PostGroup, error) {
	data, err := c.request("GET", "/get-post/"+postGroupID, nil)
	if err != nil {
		return nil, err
	}

	var result PostGroup
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return &result, nil
}

func (c *Client) UpdatePost(postGroupID string, updates *CreatePostRequest) (*PostGroup, error) {
	data, err := c.request("PUT", "/update-post/"+postGroupID, updates)
	if err != nil {
		return nil, err
	}

	var result PostGroup
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return &result, nil
}

func (c *Client) DeletePost(postGroupID string) error {
	_, err := c.request("DELETE", "/delete-post/"+postGroupID, nil)
	return err
}

func (c *Client) GetUploadURL(fileName, mimeType string) (*UploadURLResponse, error) {
	data, err := c.request("POST", "/get-upload-url", map[string]string{
		"fileName": fileName,
		"mimeType": mimeType,
	})
	if err != nil {
		return nil, err
	}

	var result UploadURLResponse
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return &result, nil
}

func (c *Client) GetLinkedInPostStats(platformConnectionID, postURN string) (*LinkedInStats, error) {
	data, err := c.request("POST", "/linkedin-post-statistics", map[string]string{
		"platformConnectionId": platformConnectionID,
		"postUrn":              postURN,
	})
	if err != nil {
		return nil, err
	}

	var result LinkedInStats
	if err := json.Unmarshal(data, &result); err != nil {
		return nil, err
	}

	return &result, nil
}
```

## Usage Examples

### List Connections

```go
package main

import (
	"fmt"
	"log"
	"os"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	connections, err := client.GetConnections()
	if err != nil {
		log.Fatalf("Failed to get connections: %v", err)
	}

	fmt.Printf("Found %d connected accounts:\n", len(connections))
	for _, conn := range connections {
		fmt.Printf("  - %s: %s (%s)\n", conn.Platform, conn.Username, conn.PlatformID)
	}
}
```

### Create a Post

```go
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	// Schedule for 1 hour from now
	scheduledTime := time.Now().Add(time.Hour).UTC().Format(time.RFC3339)

	resp, err := client.CreatePost(&publora.CreatePostRequest{
		Content:       "Hello from Go! Building with Publora API.",
		Platforms:     []string{"twitter-123456789", "linkedin-ABC123DEF"},
		ScheduledTime: scheduledTime,
	})
	if err != nil {
		log.Fatalf("Failed to create post: %v", err)
	}

	fmt.Printf("Post created: %s\n", resp.PostGroupID)
	for _, post := range resp.Posts {
		fmt.Printf("  - %s: %s\n", post.Platform, post.Status)
	}
}
```

### Schedule Multiple Posts

```go
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"your-project/publora"
)

type ScheduledPost struct {
	Content   string
	Platforms []string
	DayOffset int
}

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	posts := []ScheduledPost{
		{"Monday motivation: Start strong!", []string{"twitter-123456789"}, 0},
		{"Tech tip Tuesday: Version your APIs!", []string{"twitter-123456789", "linkedin-ABC123DEF"}, 1},
		{"Wednesday wisdom: Ship fast, iterate faster.", []string{"twitter-123456789"}, 2},
		{"Throwback Thursday: How we grew to 10K users.", []string{"linkedin-ABC123DEF"}, 3},
		{"Feature Friday: Check out our new dashboard!", []string{"twitter-123456789", "linkedin-ABC123DEF"}, 4},
	}

	baseTime := time.Now().Truncate(24 * time.Hour).Add(9 * time.Hour) // 9 AM

	for _, post := range posts {
		scheduledTime := baseTime.AddDate(0, 0, post.DayOffset).Format(time.RFC3339)

		resp, err := client.CreatePost(&publora.CreatePostRequest{
			Content:       post.Content,
			Platforms:     post.Platforms,
			ScheduledTime: scheduledTime,
		})
		if err != nil {
			log.Printf("Failed to schedule post: %v", err)
			continue
		}

		fmt.Printf("Scheduled: %s -> %s\n", post.Content[:30], resp.PostGroupID)
		time.Sleep(200 * time.Millisecond) // Rate limiting
	}
}
```

### Post with Instagram Reel Settings

```go
package main

import (
	"fmt"
	"log"
	"os"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	resp, err := client.CreatePost(&publora.CreatePostRequest{
		Content:   "Behind the scenes! #buildinpublic",
		Platforms: []string{"instagram-789012345"},
		MediaURLs: []string{"https://example.com/video.mp4"},
		PlatformSettings: &publora.PlatformSettings{
			Instagram: &publora.InstagramSettings{
				VideoType: "REELS",
			},
		},
	})
	if err != nil {
		log.Fatalf("Failed to create reel: %v", err)
	}

	fmt.Printf("Instagram Reel scheduled: %s\n", resp.PostGroupID)
}
```

### Upload Media and Post

```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"path/filepath"

	"your-project/publora"
)

func getMimeType(filename string) string {
	ext := filepath.Ext(filename)
	mimeTypes := map[string]string{
		".jpg":  "image/jpeg",
		".jpeg": "image/jpeg",
		".png":  "image/png",
		".gif":  "image/gif",
		".mp4":  "video/mp4",
		".webp": "image/webp",
	}
	if mime, ok := mimeTypes[ext]; ok {
		return mime
	}
	return "application/octet-stream"
}

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))
	filePath := "./screenshot.png"

	// Step 1: Get upload URL
	fileName := filepath.Base(filePath)
	mimeType := getMimeType(fileName)

	uploadResp, err := client.GetUploadURL(fileName, mimeType)
	if err != nil {
		log.Fatalf("Failed to get upload URL: %v", err)
	}

	// Step 2: Upload file to S3
	fileData, err := os.ReadFile(filePath)
	if err != nil {
		log.Fatalf("Failed to read file: %v", err)
	}

	req, _ := http.NewRequest("PUT", uploadResp.UploadURL, bytes.NewReader(fileData))
	req.Header.Set("Content-Type", mimeType)

	httpClient := &http.Client{}
	resp, err := httpClient.Do(req)
	if err != nil {
		log.Fatalf("Failed to upload file: %v", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode >= 400 {
		body, _ := io.ReadAll(resp.Body)
		log.Fatalf("Upload failed: %s", body)
	}

	// Step 3: Create post with media
	postResp, err := client.CreatePost(&publora.CreatePostRequest{
		Content:   "Check out this screenshot!",
		Platforms: []string{"twitter-123456789", "linkedin-ABC123DEF"},
		MediaKeys: []string{uploadResp.MediaKey},
	})
	if err != nil {
		log.Fatalf("Failed to create post: %v", err)
	}

	fmt.Printf("Post with media created: %s\n", postResp.PostGroupID)
}
```

### Error Handling

```go
package main

import (
	"errors"
	"fmt"
	"log"
	"os"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	_, err := client.CreatePost(&publora.CreatePostRequest{
		Content:   "Test post",
		Platforms: []string{"twitter-123456789"},
	})

	if err != nil {
		var apiErr *publora.APIError
		if errors.As(err, &apiErr) {
			switch apiErr.StatusCode {
			case 401:
				fmt.Println("Authentication failed. Check your API key.")
			case 403:
				fmt.Printf("Access denied: %s\n", apiErr.Message)
			case 400:
				fmt.Printf("Bad request: %s\n", apiErr.Message)
			case 404:
				fmt.Printf("Not found: %s\n", apiErr.Message)
			case 429:
				fmt.Println("Rate limited. Retry after delay.")
			case 500:
				fmt.Println("Server error. Retry later.")
			default:
				fmt.Printf("API error (%d): %s\n", apiErr.StatusCode, apiErr.Message)
			}
		} else {
			log.Fatalf("Network error: %v", err)
		}
		return
	}

	fmt.Println("Post created successfully!")
}
```

### Monitor Post Status

```go
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"your-project/publora"
)

func waitForPublish(client *publora.Client, postGroupID string, maxAttempts int) (*publora.PostGroup, error) {
	for attempt := 0; attempt < maxAttempts; attempt++ {
		postGroup, err := client.GetPost(postGroupID)
		if err != nil {
			return nil, err
		}

		fmt.Printf("[%d/%d] Status: %s\n", attempt+1, maxAttempts, postGroup.Status)

		switch postGroup.Status {
		case "published":
			fmt.Println("All platforms published successfully!")
			return postGroup, nil
		case "partially_published":
			fmt.Println("Partial success:")
			for _, post := range postGroup.Posts {
				if post.Status == "failed" {
					fmt.Printf("  - %s: FAILED - %s\n", post.Platform, post.Error)
				} else {
					fmt.Printf("  - %s: %s\n", post.Platform, post.Status)
				}
			}
			return postGroup, nil
		case "failed":
			fmt.Println("All platforms failed:")
			for _, post := range postGroup.Posts {
				fmt.Printf("  - %s: %s\n", post.Platform, post.Error)
			}
			return postGroup, nil
		}

		time.Sleep(10 * time.Second)
	}

	return nil, fmt.Errorf("timeout waiting for post %s to publish", postGroupID)
}

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	resp, err := client.CreatePost(&publora.CreatePostRequest{
		Content:   "Publishing now!",
		Platforms: []string{"twitter-123456789"},
	})
	if err != nil {
		log.Fatalf("Failed to create post: %v", err)
	}

	postGroup, err := waitForPublish(client, resp.PostGroupID, 30)
	if err != nil {
		log.Fatalf("Failed to monitor post: %v", err)
	}

	fmt.Printf("Final status: %s\n", postGroup.Status)
}
```

### Concurrent Batch Operations

```go
package main

import (
	"fmt"
	"os"
	"sync"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	postGroupIDs := []string{"pg_abc123", "pg_def456", "pg_ghi789", "pg_jkl012"}

	var wg sync.WaitGroup
	results := make(chan struct {
		id     string
		status string
		err    error
	}, len(postGroupIDs))

	// Limit concurrency
	sem := make(chan struct{}, 3)

	for _, id := range postGroupIDs {
		wg.Add(1)
		go func(postGroupID string) {
			defer wg.Done()
			sem <- struct{}{}
			defer func() { <-sem }()

			postGroup, err := client.GetPost(postGroupID)
			if err != nil {
				results <- struct {
					id     string
					status string
					err    error
				}{postGroupID, "", err}
				return
			}

			results <- struct {
				id     string
				status string
				err    error
			}{postGroupID, postGroup.Status, nil}
		}(id)
	}

	go func() {
		wg.Wait()
		close(results)
	}()

	for result := range results {
		if result.err != nil {
			fmt.Printf("%s: ERROR - %v\n", result.id, result.err)
		} else {
			fmt.Printf("%s: %s\n", result.id, result.status)
		}
	}
}
```

### LinkedIn Analytics

```go
package main

import (
	"fmt"
	"log"
	"os"

	"your-project/publora"
)

func main() {
	client := publora.NewClient(os.Getenv("PUBLORA_API_KEY"))

	platformConnectionID := "linkedin-ABC123DEF"
	postURNs := []string{
		"urn:li:share:7123456789012345678",
		"urn:li:share:7234567890123456789",
	}

	fmt.Println("=== LinkedIn Analytics Report ===")

	var totalImpressions, totalEngagement int

	for _, postURN := range postURNs {
		stats, err := client.GetLinkedInPostStats(platformConnectionID, postURN)
		if err != nil {
			log.Printf("Error fetching stats for %s: %v", postURN, err)
			continue
		}

		engagement := stats.Likes + stats.Comments + stats.Shares
		totalImpressions += stats.Impressions
		totalEngagement += engagement

		fmt.Printf("\nPost: %s\n", postURN)
		fmt.Printf("  Impressions: %d\n", stats.Impressions)
		fmt.Printf("  Engagement: %d (%d likes, %d comments, %d shares)\n",
			engagement, stats.Likes, stats.Comments, stats.Shares)
		fmt.Printf("  CTR: %.2f%%\n", float64(stats.Clicks)/float64(stats.Impressions)*100)
	}

	fmt.Println("\n=== TOTALS ===")
	fmt.Printf("Total Impressions: %d\n", totalImpressions)
	fmt.Printf("Total Engagement: %d\n", totalEngagement)
	if totalImpressions > 0 {
		fmt.Printf("Avg Engagement Rate: %.2f%%\n", float64(totalEngagement)/float64(totalImpressions)*100)
	}
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
