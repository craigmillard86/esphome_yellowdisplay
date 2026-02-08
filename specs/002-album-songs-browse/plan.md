# Implementation Plan: Album Songs Browse

**Branch**: `002-album-songs-browse` | **Date**: 2026-02-08 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/002-album-songs-browse/spec.md`

## Summary

Add album songs browsing capability to the Music Remote: when users tap on an album (from Artists → Artist Albums or from Albums browse), display the album's songs with track number and title. Users can tap a song to play it or use "Play All" to play the entire album.

## Technical Context

**Language/Version**: ESPHome YAML with embedded C++ lambdas
**Primary Dependencies**: LVGL (via ESPHome), Home Assistant Native API, Music Assistant
**Storage**: NVS for browse state (existing pattern)
**Testing**: Manual testing on device, ESPHome compile validation
**Target Platform**: ESP32-2432S028R (CYD) and ESP32-S3 (Freenove)
**Project Type**: ESPHome embedded device
**Performance Goals**: Song list loads in <2 seconds, tap response <100ms
**Constraints**: ~320KB RAM, 4 items per page (existing pagination), 320×240 display
**Scale/Scope**: Albums typically have 10-20 songs, paginated in groups of 4

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Status | Notes |
|------|--------|-------|
| §1.1 Reliability | ✅ Pass | Uses existing browse infrastructure with error handling |
| §1.2 UI Responsiveness | ✅ Pass | Reuses pagination pattern, no blocking calls |
| §1.3 Deterministic Behavior | ✅ Pass | New category ID (7 = AlbumSongs) follows existing pattern |
| §2.1 YAML vs Custom Code | ✅ Pass | Pure ESPHome YAML, no custom C++ components |
| §3.1 Modularization | ✅ Pass | Extends existing browse.yaml and browse_unified.yaml |
| §3.2 Clean Code | ✅ Pass | Follows existing naming conventions |
| §6 Naming Conventions | ✅ Pass | Uses existing browse_* ID patterns |
| §9.2 Offline Behavior | ✅ Pass | Inherits existing offline handling |

## Project Structure

### Documentation (this feature)

```text
specs/002-album-songs-browse/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (files to modify/create)

```text
esphome/
├── packages/
│   ├── browse.yaml          # MODIFY: Add category 7 (AlbumSongs), browse_album_songs script
│   └── navigation.yaml      # MODIFY: Add screen 13 (BROWSE_ALBUM_SONGS) if separate page needed
└── ui/
    └── browse_unified.yaml  # MODIFY: Update item tap handlers for album drill-down
```

**Structure Decision**: Reuse the existing unified browse page (`page_browse`) with a new category (7 = AlbumSongs). This follows the Artist Albums pattern (category 6) and minimizes memory footprint by not adding a new LVGL page.

## Complexity Tracking

No constitution violations requiring justification.

## Design Approach

### Category Mapping (Extended)

| ID | Category | Title | Alpha Picker | Item Tap Action |
|----|----------|-------|--------------|-----------------|
| 1 | Recent | "Recently Played" | No | Play media |
| 2 | Favorites | "Favorites" | No | Play media |
| 3 | Playlists | "Playlists" | No | Play media |
| 4 | Albums | "Albums" | Yes | Browse album songs (NEW) |
| 5 | Artists | "Artists" | Yes | Browse artist albums |
| 6 | ArtistAlbums | "[Artist Name]" | No | Browse album songs (NEW) |
| 7 | **AlbumSongs** | **"[Album Name]"** | **No** | **Play song** |

### Navigation Flow

```
Library → Albums → [Album] → Album Songs → [Song] → Now Playing
Library → Artists → [Artist] → Artist Albums → [Album] → Album Songs → [Song] → Now Playing
```

### Data Flow for Album Songs

1. User taps album in Albums (cat 4) or Artist Albums (cat 6)
2. `browse_album_songs` script saves album context (id, name)
3. Configure page for category 7 (title = album name, no alpha picker, show Play All)
4. Query MA: `music_assistant.get_library` with `media_type: track`, `album: [album_id]`
5. Display songs with format "1. Song Name"
6. Item tap → `play_media_item` → navigate to Now Playing

### Context Variables (New)

```yaml
globals:
  - id: browse_context_album_id
    type: std::string
  - id: browse_context_album_name
    type: std::string
```

### Back Navigation

From Album Songs (category 7):
- If came from Albums (cat 4): restore to Albums at saved offset
- If came from Artist Albums (cat 6): restore to Artist Albums at saved offset

Use pattern: save `browse_saved_offset` and `browse_saved_category` before drill-down.

### Music Assistant API

Query album tracks:
```yaml
http_request.post:
  url: "${ha_api_url}/api/services/music_assistant/get_library?return_response"
  json:
    media_type: "track"
    album: !lambda 'return id(browse_context_album_id).c_str();'
    limit: 4
    offset: !lambda 'return id(browse_page_offset);'
    order_by: "track_number"
    config_entry_id: "${ma_config_entry_id}"
```

### Song Display Format

Per clarification: "Track number + title only"
```cpp
// Format: "1. Song Name"
char display[64];
snprintf(display, sizeof(display), "%d. %s", track_number, title);
```

## Implementation Tasks (High-Level)

1. **Add Album Context Variables** - New globals for album_id and album_name
2. **Add Category 7 Handler** - In `browse_configure_page` and `browse_category`
3. **Create `browse_album_songs` Script** - Saves context, configures page, loads songs
4. **Update Item Tap Handlers** - Categories 4 and 6 now drill into album songs
5. **Add Category 7 to `browse_load_page`** - Parse track response, format display
6. **Update Back Navigation** - Handle return from category 7 to 4 or 6
7. **Update Play All** - Works for category 7 (plays album via album_id)
8. **Test All Navigation Paths** - Albums → Songs, Artists → Albums → Songs

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| MA API doesn't return track number | Low | Medium | Fall back to list order (1, 2, 3...) |
| Memory pressure from nested navigation | Low | High | Reuse existing page, clear previous data |
| Back navigation state confusion | Medium | Medium | Clear separation of saved_category/saved_offset |

## Next Steps

1. Run `/speckit.tasks` to generate detailed implementation tasks
2. Implement changes incrementally, testing each step
3. Compile test on both CYD and Freenove S3 targets
