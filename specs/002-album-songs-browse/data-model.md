# Data Model: Album Songs Browse

**Feature**: 002-album-songs-browse
**Date**: 2026-02-08

## Entities

### Album (Existing, Extended Context)

| Field | Type | Source | Notes |
|-------|------|--------|-------|
| `browse_context_album_id` | std::string | MA API `uri` | e.g., "library://album/123" |
| `browse_context_album_name` | std::string | MA API `name` | Album title for header display |

### Track/Song (New Entity)

| Field | Type | Source | Notes |
|-------|------|--------|-------|
| `media_id` | std::string | MA API `uri` | e.g., "library://track/456" |
| `title` | std::string | MA API `name` | Song title |
| `track_number` | int | MA API `track_number` | Track position, fallback to index |
| `subtitle` | std::string | Derived | Empty for songs (no artist shown) |

### Browse State (Extended)

| Field | Type | Purpose |
|-------|------|---------|
| `current_browse_category` | int | Now includes 7=AlbumSongs |
| `browse_saved_category` | int | Parent category for back nav (4 or 6) |
| `browse_saved_offset` | int | Page offset before drill-down (existing) |

## State Transitions

### Category 7 (AlbumSongs) Entry

```
From Category 4 (Albums):
  1. User taps album item
  2. Save: browse_saved_category = 4, browse_saved_offset = current offset
  3. Set: browse_context_album_id, browse_context_album_name
  4. Set: current_browse_category = 7, browse_page_offset = 0
  5. Load album tracks

From Category 6 (ArtistAlbums):
  1. User taps album item
  2. Save: browse_saved_category = 6, browse_saved_offset = current offset
  3. Set: browse_context_album_id, browse_context_album_name
  4. Set: current_browse_category = 7, browse_page_offset = 0
  5. Load album tracks
```

### Category 7 (AlbumSongs) Exit

```
Back Button:
  1. Restore: current_browse_category = browse_saved_category
  2. Restore: browse_page_offset = browse_saved_offset
  3. Clear: browse_context_album_id, browse_context_album_name
  4. Reload parent category at saved offset
```

## Display Format

### Song List Item

```
+----------------------------------------+
| 1. Song Title                           |
|                                         |
+----------------------------------------+
```

- Title format: "{track_number}. {song_title}"
- Subtitle: Empty (hidden or blank)
- Height: 36px (standard list item)
- Tap action: Play song

### Page Header (Category 7)

```
+----------------------------------------+
| [<] Album Name             [Play All]   |
+----------------------------------------+
```

- Back button: Returns to parent category
- Title: Album name from context
- Play All: Visible, plays entire album

## Validation Rules

1. **track_number**: Must be >= 1 (use index+1 if missing from API)
2. **browse_context_album_id**: Must not be empty when category = 7
3. **browse_saved_category**: Must be 4 or 6 when returning from category 7
