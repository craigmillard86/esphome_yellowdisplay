# API Contracts: Artist Album Browse

**Feature**: 001-artist-album-browse
**Date**: 2026-02-07

## Overview

This feature uses existing Music Assistant API endpoints via Home Assistant REST API. No new endpoints are required.

## Endpoints

### 1. Get Albums by Artist

**Endpoint**: `POST /api/services/music_assistant/get_library`
**Authorization**: Bearer token (long-lived HA token)

#### Request

```json
{
  "media_type": "album",
  "artist": "library://artist/123",
  "limit": 4,
  "offset": 0,
  "order_by": "name",
  "config_entry_id": "<ma_config_entry_id>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `media_type` | string | Yes | Must be `"album"` |
| `artist` | string | Yes | Artist URI to filter by |
| `limit` | int | Yes | Page size (4 for this device) |
| `offset` | int | Yes | Pagination offset |
| `order_by` | string | No | Sort order, default `"name"` |
| `config_entry_id` | string | Yes | Music Assistant config entry ID |

#### Response

```json
{
  "service_response": {
    "items": [
      {
        "uri": "library://album/456",
        "name": "Album Title",
        "year": 2023,
        "artist": {
          "name": "Artist Name"
        }
      }
    ],
    "total": 12
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `items` | array | Album objects |
| `items[].uri` | string | Album media URI for playback |
| `items[].name` | string | Album title |
| `items[].year` | int | Release year (optional) |
| `total` | int | Total albums for this artist |

#### Error Responses

| Code | Description |
|------|-------------|
| 401 | Invalid or expired token |
| 404 | Artist not found |
| 500 | Music Assistant unavailable |

---

### 2. Play Artist (All Tracks)

**Endpoint**: `POST /api/services/music_assistant/play_media`
**Authorization**: Bearer token (long-lived HA token)

#### Request

```json
{
  "entity_id": "media_player.kitchen_speaker",
  "media_id": "library://artist/123"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity_id` | string | Yes | Target media player |
| `media_id` | string | Yes | Artist URI to play |

#### Response

```json
{
  "success": true
}
```

Service call returns immediately. Playback state changes asynchronously.

#### Error Responses

| Code | Description |
|------|-------------|
| 401 | Invalid or expired token |
| 404 | Player or artist not found |
| 503 | Player unavailable |

---

### 3. Play Album (Existing)

Same as Play Artist, but with album URI:

```json
{
  "entity_id": "media_player.kitchen_speaker",
  "media_id": "library://album/456"
}
```

---

## URI Formats

| Type | Format | Example |
|------|--------|---------|
| Artist | `library://artist/{id}` | `library://artist/123` |
| Album | `library://album/{id}` | `library://album/456` |
| Track | `library://track/{id}` | `library://track/789` |

---

## Implementation Notes

### HTTP Configuration

```yaml
http_request:
  useragent: "ESPHome CYD Music Remote"
  timeout: 15s
  verify_ssl: false  # Local HA uses self-signed certs
```

### Request Headers

```cpp
// Set in HTTP request
request->set_header("Authorization", ("Bearer " + token).c_str());
request->set_header("Content-Type", "application/json");
```

### Response Parsing

The device uses manual JSON parsing (no library) due to memory constraints:

```cpp
// Extract field from JSON string
std::string extract_json_string(const std::string& json, const std::string& key) {
  size_t pos = json.find("\"" + key + "\"");
  // ... parsing logic
}
```

### Error Handling

All API calls should:
1. Check HTTP status code
2. Validate response contains expected fields
3. Log errors with `ESP_LOGW` or `ESP_LOGE`
4. Set `browse_loading = false` on failure
5. Display user-friendly error message in UI
