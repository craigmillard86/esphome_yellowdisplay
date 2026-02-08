# Research: Album Songs Browse

**Feature**: 002-album-songs-browse
**Date**: 2026-02-08

## Research Questions

### Q1: How does Music Assistant API return track information?

**Decision**: Use `music_assistant.get_library` service with `media_type: track` and `album` filter

**Rationale**:
- Existing browse implementation uses this pattern for other media types
- The `album` parameter filters tracks to a specific album
- `order_by: track_number` ensures correct ordering

**Alternatives Considered**:
- Direct MA REST API call - More complex, would need separate authentication
- Template query - Less efficient, harder to paginate

**API Response Format** (based on existing patterns):
```json
{
  "service_response": {
    "items": [
      {
        "uri": "library://track/123",
        "name": "Song Title",
        "track_number": 1,
        "artists": [{"name": "Artist Name"}]
      }
    ],
    "total": 12
  }
}
```

### Q2: How to handle track number extraction?

**Decision**: Parse `track_number` field from API response, fall back to index+1 if missing

**Rationale**:
- MA API includes track_number in track metadata
- Some imported tracks may lack this field
- Using list index as fallback ensures display consistency

**Implementation**:
```cpp
// Extract track_number or use index+1
std::string track_search = "\"track_number\":";
size_t num_pos = body.find(track_search, item_start);
int track_num = (num_pos != std::string::npos && num_pos < item_end)
                ? atoi(body.c_str() + num_pos + track_search.length())
                : item_count + 1;
```

### Q3: How to extend navigation for deeper drill-down?

**Decision**: Add `browse_saved_category` global to track parent category for back navigation

**Rationale**:
- Current implementation only tracks single drill-down (Artists → Artist Albums)
- Need to support two-level: Albums → Album Songs AND Artists → Artist Albums → Album Songs
- Saving parent category enables correct back navigation

**Variables**:
```yaml
globals:
  - id: browse_saved_category
    type: int
    restore_value: no
    initial_value: '0'
```

### Q4: Should Play All use album URI or queue individual tracks?

**Decision**: Play album URI directly (same as existing artist Play All)

**Rationale**:
- MA handles album playback internally, ensuring correct track order
- Simpler implementation - single service call
- Consistent with existing `play_artist_all` pattern

**Implementation**:
```yaml
- homeassistant.service:
    service: music_assistant.play_media
    data:
      entity_id: !lambda 'return id(selected_player_id);'
      media_id: !lambda 'return id(browse_context_album_id);'
```

### Q5: How to handle empty albums?

**Decision**: Show "No songs in this album" empty state

**Rationale**:
- Matches existing empty state pattern in browse
- Provides clear user feedback
- Edge case but should be handled gracefully

**Implementation**: Reuse existing `browse_empty_state` widget

## Technical Findings

### Current Browse Category Mapping
- 1: Recent
- 2: Favorites
- 3: Playlists
- 4: Albums (will change: tap → album songs instead of play)
- 5: Artists
- 6: ArtistAlbums (will change: tap → album songs instead of play)
- **7: AlbumSongs (NEW)**

### Navigation Stack Depth
Current max depth is 3 (`nav_stack_max_depth`). Navigation paths:
- Albums → AlbumSongs = 2 levels (OK)
- Artists → ArtistAlbums → AlbumSongs = 3 levels (OK, at limit)

No change needed to stack depth.

### Memory Considerations
- Reusing `page_browse` means no additional LVGL page allocation
- Track data replaces album/artist data in same globals
- 4 items per page with existing pagination

## Conclusion

All technical questions resolved. Ready for implementation with:
1. New category 7 (AlbumSongs)
2. New globals: `browse_context_album_id`, `browse_context_album_name`, `browse_saved_category`
3. New script: `browse_album_songs`
4. Modified tap handlers for categories 4 and 6
5. Modified back navigation for category 7
