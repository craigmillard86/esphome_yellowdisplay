# Research: Artist Album Browse

**Feature**: 001-artist-album-browse
**Date**: 2026-02-07

## R1: Music Assistant API - Albums by Artist

### Question
How to query albums filtered by a specific artist using the existing Music Assistant integration?

### Research Findings

The existing browse system uses `music_assistant.get_library` via Home Assistant REST API. Based on Music Assistant documentation and existing code patterns:

**Current Album Query** (browse.yaml:447-470):
```cpp
root["media_type"] = "album";
root["limit"] = 4;
root["offset"] = browse_page_offset;
root["order_by"] = "name";
```

**Artist-Filtered Album Query** (new):
```cpp
root["media_type"] = "album";
root["artist"] = id(browse_context_artist_id).c_str();  // Artist URI filter
root["limit"] = 4;
root["offset"] = browse_page_offset;
root["order_by"] = "name";
```

The `artist` parameter accepts the artist URI (e.g., `library://artist/123`) and filters returned albums to only those by that artist.

### Decision
Use `artist` filter parameter in existing `get_library` service call.

### Rationale
- Minimal code changes required
- Uses same response parsing logic
- Consistent with existing patterns

---

## R2: Play All Artist Implementation

### Question
How to play all music by an artist with a single action?

### Research Findings

Music Assistant's `play_media` service accepts artist URIs directly. When given an artist URI, it queues all tracks by that artist.

**Existing Play Media Call** (browse.yaml:718-752):
```yaml
homeassistant.service:
  service: music_assistant.play_media
  data:
    entity_id: !lambda 'return id(selected_player_id).c_str();'
    media_id: !lambda 'return media_id.c_str();'
```

**Play Artist** (same service, artist URI):
```yaml
homeassistant.service:
  service: music_assistant.play_media
  data:
    entity_id: !lambda 'return id(selected_player_id).c_str();'
    media_id: !lambda 'return id(browse_context_artist_id).c_str();'
```

### Decision
Reuse existing `play_media` service with artist URI.

### Rationale
- No additional API endpoints required
- Music Assistant handles queue/shuffle logic
- Consistent with existing playback pattern

---

## R3: Navigation State Management

### Question
How to track artist context for drill-down navigation?

### Research Findings

Current browse system uses `current_browse_category` (int) to track category:
- 0 = None
- 1 = Recent
- 2 = Favorites
- 3 = Playlists
- 4 = Albums
- 5 = Artists

For artist drill-down, we need:
1. New category value for "Artist Albums" context
2. Storage for selected artist ID (for API filtering)
3. Storage for selected artist name (for header display)
4. Saved scroll position for back navigation

### Decision
Add category 6 = ArtistAlbums and three new globals.

### Implementation
```yaml
globals:
  - id: browse_context_artist_id
    type: std::string
    restore_value: no
    initial_value: '""'

  - id: browse_context_artist_name
    type: std::string
    restore_value: no
    initial_value: '""'

  - id: browse_saved_offset
    type: int
    restore_value: no
    initial_value: '0'
```

---

## R4: UI Layout for Play All Button

### Question
Where to place the "Play All" button and how to size it?

### Research Findings

Current browse header layout (browse_unified.yaml):
- Back button: 44x44px at (0, 0)
- Title: starts at x=44, centered vertically
- Page indicator: right-aligned

Available space for Play All:
- Header height: 44px
- Screen width: 320px
- After back button + title: ~80px available on right

### Decision
Add Play All button in header, right-aligned before page indicator.

### Layout
```
| ◄ (44px) | Title (flexible) | Play All (70px) |
```

Button specs:
- Width: 70px
- Height: 32px (smaller than 44px for visual balance)
- Position: Right-aligned with 8px padding
- Style: Primary button style (cyan)
- Icon: Play icon (mdi:play) + text or just icon

---

## R5: Existing Code Patterns

### Tap Handler Pattern
Current artists list tap handler (browse_unified.yaml:146-150):
```yaml
on_click:
  - script.execute:
      id: play_media_item
      media_id: !lambda 'return id(browse_item_0_media_id);'
      item_title: !lambda 'return id(browse_item_0_title);'
```

This needs modification to:
1. Check if in Artists category (5)
2. If so, navigate to artist albums instead of playing
3. If in Artist Albums category (6), play the album

### Back Navigation Pattern
Current back navigation (navigation.yaml):
```yaml
- script.execute: navigate_back
```

This needs modification to:
1. Detect if leaving category 6 (Artist Albums)
2. If so, restore saved offset and reload Artists list
3. Otherwise, use existing behavior
