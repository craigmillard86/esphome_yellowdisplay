# Data Model: Artist Album Browse

**Feature**: 001-artist-album-browse
**Date**: 2026-02-07

## Entity Definitions

### Browse Context (New)

Tracks the current drill-down context when viewing albums by a specific artist.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `browse_context_artist_id` | std::string | Max 100 chars | URI of selected artist (e.g., `library://artist/123`) |
| `browse_context_artist_name` | std::string | Max 50 chars | Display name of selected artist |
| `browse_saved_offset` | int | >= 0 | Saved scroll position in artist list for back navigation |

### Browse Category Constants (Extended)

| Value | Name | Description |
|-------|------|-------------|
| 0 | None | No category selected |
| 1 | Recent | Recently played tracks |
| 2 | Favorites | Favorite tracks |
| 3 | Playlists | User playlists |
| 4 | Albums | All albums (alphabetical) |
| 5 | Artists | All artists (alphabetical) |
| **6** | **ArtistAlbums** | **Albums by selected artist (NEW)** |

### Existing Browse Item Structure (Unchanged)

Each page displays up to 4 items with this structure:

| Field | Type | Example |
|-------|------|---------|
| `browse_item_N_media_id` | std::string | `library://album/456` |
| `browse_item_N_title` | std::string | `Album Title` |
| `browse_item_N_subtitle` | std::string | `2023` or artist info |

Where N = 0, 1, 2, 3 (4 items per page)

## State Transitions

### Artist Browse Flow

```
[Artists List]
     |
     | (tap artist)
     v
[Save offset to browse_saved_offset]
     |
     | (set category = 6)
     v
[Artist Albums View]
     |
     +-- (tap album) --> [Play album, navigate to Now Playing]
     |
     +-- (tap Play All) --> [Play artist, navigate to Now Playing]
     |
     | (tap back)
     v
[Restore browse_saved_offset]
     |
     | (set category = 5, reload)
     v
[Artists List at saved position]
```

### Context Lifecycle

1. **Entry to Artist Albums**:
   - `browse_context_artist_id` = tapped artist's URI
   - `browse_context_artist_name` = tapped artist's name
   - `browse_saved_offset` = current `browse_page_offset`
   - `current_browse_category` = 6

2. **During Artist Albums**:
   - Album queries filtered by `browse_context_artist_id`
   - Header displays `browse_context_artist_name`
   - Pagination uses normal `browse_page_offset` (reset to 0)

3. **Exit from Artist Albums**:
   - `browse_page_offset` = `browse_saved_offset`
   - `current_browse_category` = 5
   - Context globals cleared (optional, not strictly necessary)

## API Data Mapping

### Artist Item (category 5)

| API Response Field | Maps To |
|--------------------|---------|
| `uri` or `id` | `browse_item_N_media_id` |
| `name` | `browse_item_N_title` |
| `display_name` or empty | `browse_item_N_subtitle` |

### Album Item (category 4 or 6)

| API Response Field | Maps To |
|--------------------|---------|
| `uri` or `id` | `browse_item_N_media_id` |
| `name` or `title` | `browse_item_N_title` |
| `artist` or `year` | `browse_item_N_subtitle` |

## Memory Budget

| Component | Size | Notes |
|-----------|------|-------|
| `browse_context_artist_id` | 100 bytes | URI string |
| `browse_context_artist_name` | 50 bytes | Name string |
| `browse_saved_offset` | 4 bytes | Integer |
| **Total New Memory** | **~154 bytes** | Well within constraints |

Existing browse infrastructure: ~1.5KB for 4 items
Total browse system: ~1.7KB peak
