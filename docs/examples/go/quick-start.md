# Go Quick Start

Go syntax for the five [Core Workflows](../curl/all-endpoints.md). The canonical page owns ordering and API semantics.

## Client

```go
package publora

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "time"
)

type Client struct {
    BaseURL string
    APIKey string
    HTTP *http.Client
}

func New() *Client {
    return &Client{
        BaseURL: "https://api.publora.com/api/v1",
        APIKey: os.Getenv("PUBLORA_API_KEY"),
        HTTP: &http.Client{Timeout: 30 * time.Second},
    }
}

func (c *Client) JSON(method, path string, body any, key string, out any) error {
    payload, err := json.Marshal(body)
    if err != nil { return err }
    req, err := http.NewRequest(method, c.BaseURL+"/"+path, bytes.NewReader(payload))
    if err != nil { return err }
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("x-publora-key", c.APIKey)
    if key != "" { req.Header.Set("Idempotency-Key", key) }
    resp, err := c.HTTP.Do(req)
    if err != nil { return err }
    defer resp.Body.Close()
    if resp.StatusCode < 200 || resp.StatusCode >= 300 {
        return fmt.Errorf("Publora returned %s", resp.Status)
    }
    return json.NewDecoder(resp.Body).Decode(out)
}
```

## 1. Draft

```go
var draft map[string]any
err := client.JSON("POST", "create-post", map[string]any{
    "content": "Draft from Go",
    "platforms": []string{platformID},
}, "", &draft)
```

With no `scheduledTime`, this is intentionally a draft.

## 2. One-shot schedule

```go
scheduledTime := time.Now().UTC().Add(5 * time.Minute).Format(time.RFC3339)
var scheduled map[string]any
err := client.JSON("POST", "create-post", map[string]any{
    "content": "Scheduled from Go",
    "platforms": []string{platformID},
    "scheduledTime": scheduledTime,
}, requestID, &scheduled)
```

## 3. Upload and schedule

Follow the canonical [presigned sequence](../curl/all-endpoints.md#3-upload-and-schedule). PUT the file to `uploadUrl` with a plain `http.Request` and its media content type; do not add the Publora API key to the storage request. Then:

```go
var completed map[string]any
err = client.JSON("POST", "complete-media/"+mediaFileID, nil, "", &completed)

var updated map[string]any
err = client.JSON("PUT", "update-post/"+postGroupID, map[string]any{
    "status": "scheduled",
    "scheduledTime": time.Now().UTC().Add(5 * time.Minute).Format(time.RFC3339),
}, "schedule-"+postGroupID, &updated)
```

## 4. Update

```go
err = client.JSON("PUT", "update-post/"+postGroupID, map[string]any{
    "status": "scheduled",
    "scheduledTime": time.Now().UTC().Add(10 * time.Minute).Format(time.RFC3339),
}, "reschedule-"+postGroupID, &updated)
```

## 5. Webhook consumer

```go
func webhook(secret []byte) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        raw, err := io.ReadAll(r.Body)
        if err != nil { http.Error(w, "bad body", 400); return }
        mac := hmac.New(sha256.New, secret)
        mac.Write(raw)
        received, err := hex.DecodeString(r.Header.Get("x-publora-signature"))
        if err != nil || !hmac.Equal(received, mac.Sum(nil)) {
            http.Error(w, "invalid signature", 401)
            return
        }
        var envelope map[string]any
        if json.Unmarshal(raw, &envelope) != nil { http.Error(w, "bad json", 400); return }
        w.WriteHeader(http.StatusNoContent)
    }
}
```

Add `crypto/hmac`, `crypto/sha256`, `encoding/hex`, and `io` to the imports used by the webhook file. See [Webhooks](../../endpoints/webhooks.md). Batch work and LinkedIn analytics remain discoverable through the [OpenAPI document](https://docs.publora.com/openapi.yaml), without duplicating their contract here.
