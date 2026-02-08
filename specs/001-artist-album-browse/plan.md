# Implementation Plan: Artist Album Browse

**Branch**: `001-artist-album-browse` | **Date**: 2026-02-07 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-artist-album-browse/spec.md`

## Summary

Add hierarchical artist/album browsing: tapping an artist navigates to their album list instead of playing, with a "Play All" header button and tap-to-play on individual albums. Extends existing browse infrastructure with new context mode and navigation state.

## Technical Context

**Language/Version**: ESPHome YAML with embedded C++ lambdas
**Primary Dependencies**: LVGL (via ESPHome), Home Assistant Native API, Music Assistant
**Storage**: NVS for persisted state (existing pattern)
**Testing**: Manual functional testing per Constitution §11.2
**Target Platform**: ESP32-2432S028R (CYD) @ 240MHz, ~50KB free heap
**Project Type**: Embedded firmware (ESPHome packages)
**Performance Goals**: Navigation <2s, touch feedback <100ms per Constitution §1.2
**Constraints**: 4 items per page (memory), 320x240 display, resistive touch
**Scale/Scope**: Libraries of 500+ artists, 100+ albums per artist

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Section | Requirement | Status |
|---------|-------------|--------|
| §1.1 Reliability | Graceful handling of API failures | ✅ Uses existing error handling |
| §1.2 UI Responsiveness | Touch feedback <100ms | ✅ Existing button styles |
| §1.3 Deterministic Behavior | Explicit entity/object IDs | ✅ Will follow naming conventions |
| §2.1 YAML vs Custom | ESPHome YAML primary | ✅ No custom C++ needed |
| §3.1 Modularization | Single responsibility packages | ✅ Extends browse.yaml |
| §3.2 Clean Code | No magic numbers, DRY | ✅ Will use substitutions |
| §6.3 LVGL Object IDs | Pattern: ui_page_component_role | ✅ Will follow pattern |
| §7.1 Display Performance | Partial invalidation, rate limiting | ✅ Uses existing patterns |
| §7.2 Resistive Touch | 44x44px minimum targets | ✅ Existing item size |
| §9.2 Offline Behavior | Controls disabled when offline | ✅ Existing gating |
| §11.3 Stability Gates | No memory leaks | ⚠️ Monitor heap usage |

## Project Structure

### Documentation (this feature)

```text
specs/001-artist-album-browse/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
esphome/
├── main.yaml                    # Entry point (no changes expected)
├── packages/
│   ├── browse.yaml              # MODIFY: Add artist album context, API calls
│   └── navigation.yaml          # MODIFY: Add artist→albums navigation
└── ui/
    └── browse_unified.yaml      # MODIFY: Add Play All button, context switching
```

**Structure Decision**: All changes extend existing ESPHome package structure. No new files required - modifications to browse.yaml, navigation.yaml, and browse_unified.yaml.

## Complexity Tracking

| Deviation | Why Needed | Simpler Alternative Rejected |
|-----------|------------|------------------------------|
| None | — | — |

No constitutional violations. Design uses existing patterns.

---

## Phase 0: Research Summary

### R1: Music Assistant API - Albums by Artist

**Decision**: Use `music_assistant.get_library` service with `media_type: album` and `artist` filter parameter

**Rationale**:
- Music Assistant API supports `artist` parameter in get_library calls
- Format: `{"media_type": "album", "artist": "<artist_uri>", "limit": 4, "offset": 0}`
- Returns same structure as current album browse: `{"service_response":{"items":[...],"total":N}}`

**Alternatives Considered**:
- Direct REST API to MA server: Rejected (would require additional secrets/config)
- Search API: Rejected (less precise than library query with artist filter)

### R2: Play All Artist Implementation

**Decision**: Use `music_assistant.play_media` with artist URI directly

**Rationale**:
- Music Assistant supports playing artist URIs directly
- Service call: `{"entity_id": "<player>", "media_id": "<artist_uri>"}`
- MA handles shuffle/queue logic internally

**Alternatives Considered**:
- Queue all albums individually: Rejected (complex, slow)
- Create temporary playlist: Rejected (requires additional API calls)

### R3: Navigation State Management

**Decision**: Add `browse_artist_context` globals to track drill-down state

**Rationale**:
- Current system uses `current_browse_category` (int) for category type
- Add new category value: 6 = Artist Albums
- Store selected artist ID and name in new globals for header display and API calls

**Implementation Details**:
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
```

### R4: Back Navigation with Scroll Position

**Decision**: Store artist list offset before drill-down, restore on back

**Rationale**:
- Constitution §11.2 requires responsive navigation
- Existing `browse_page_offset` can be saved to temp variable
- On back: restore offset and call `browse_load_page` with saved position

---

## Phase 1: Design

### Data Model

See [data-model.md](data-model.md) for entity definitions.

**Key Additions**:

| Global ID | Type | Purpose |
|-----------|------|---------|
| `browse_context_artist_id` | std::string | Selected artist's URI for API filtering |
| `browse_context_artist_name` | std::string | Selected artist's name for header display |
| `browse_saved_offset` | int | Saved scroll position for back navigation |

**Category Constants**:
- Existing: 1=Recent, 2=Favorites, 3=Playlists, 4=Albums, 5=Artists
- New: 6=ArtistAlbums (drill-down from artist)

### UI Layout Changes

**Artist Albums Screen** (browse_unified.yaml modifications):

```
┌─────────────────────────────────────────────────┐
│ ◄  Artist Name                    [▶ Play All] │  <- Modified header
├─────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────┐ │
│ │ Album 1 Title                               │ │
│ │ Year or Info                                │ │
│ ├─────────────────────────────────────────────┤ │
│ │ Album 2 Title                               │ │  <- No alpha picker
│ │ Year or Info                                │ │     (filtered by artist)
│ ├─────────────────────────────────────────────┤ │
│ │ Album 3 Title                               │ │
│ │ Year or Info                                │ │
│ ├─────────────────────────────────────────────┤ │
│ │ Album 4 Title                               │ │
│ │ Year or Info                                │ │
│ └─────────────────────────────────────────────┘ │
│  < Prev      1-4 of 12      Next >              │
└─────────────────────────────────────────────────┘
```

**Header Changes**:
- When `current_browse_category == 6`: Show artist name + Play All button
- Play All button: 60x32px, positioned right of title
- Back button behavior: Return to Artists list (category 5)

### Script Changes

**browse.yaml Modifications**:

1. **New Script: `browse_artist_albums`**
   - Parameters: `artist_id`, `artist_name`
   - Saves current offset to `browse_saved_offset`
   - Sets context globals
   - Sets `current_browse_category = 6`
   - Calls `browse_load_page` with offset 0

2. **Modify: `browse_load_page`**
   - Add case for category 6
   - API call includes `artist` filter parameter
   - Request: `{"media_type": "album", "artist": artist_id, ...}`

3. **New Script: `play_artist_all`**
   - Parameters: none (uses `browse_context_artist_id`)
   - Calls `music_assistant.play_media` with artist URI
   - Navigates to Now Playing screen

4. **Modify: `browse_item_tap`** (or item click handlers)
   - When in category 5 (Artists): Call `browse_artist_albums` instead of `play_media_item`
   - When in category 6 (Artist Albums): Call `play_media_item` (existing behavior)

**navigation.yaml Modifications**:

1. **Modify: `navigate_back`**
   - When leaving category 6: Restore `browse_saved_offset`
   - Set category back to 5 (Artists)
   - Reload artists page at saved offset

### API Contract

**Get Albums by Artist**:
```json
POST /api/services/music_assistant/get_library
{
  "media_type": "album",
  "artist": "library://artist/123",
  "limit": 4,
  "offset": 0,
  "order_by": "name",
  "config_entry_id": "<ma_config_entry_id>"
}

Response:
{
  "service_response": {
    "items": [
      {
        "uri": "library://album/456",
        "name": "Album Title",
        "year": 2023
      }
    ],
    "total": 12
  }
}
```

**Play Artist**:
```json
POST /api/services/music_assistant/play_media
{
  "entity_id": "media_player.kitchen_speaker",
  "media_id": "library://artist/123"
}
```

### Memory Impact Analysis

| Component | Memory Cost | Notes |
|-----------|-------------|-------|
| `browse_context_artist_id` | ~100 bytes | URI string |
| `browse_context_artist_name` | ~50 bytes | Artist name |
| `browse_saved_offset` | 4 bytes | Integer |
| Play All button | ~200 bytes | LVGL widget |
| **Total** | ~350 bytes | Minimal impact |

### Error Handling

| Scenario | Handling |
|----------|----------|
| Artist has 0 albums | Show "No albums" message, Play All still functional |
| API call fails | Show existing error state, allow retry |
| Playback fails | Log error, UI remains on album list |
| Back pressed rapidly | Debounce via existing navigation guard |

---

## Verification Plan

### Manual Testing Checklist

1. **P1 - Browse Albums by Artist**
   - [ ] Navigate to Artists screen
   - [ ] Tap an artist with multiple albums
   - [ ] Verify albums screen loads within 2 seconds
   - [ ] Verify header shows artist name
   - [ ] Verify pagination works (if >4 albums)
   - [ ] Tap back button, verify return to Artists list
   - [ ] Verify scroll position restored

2. **P2 - Play All**
   - [ ] From artist albums screen, tap Play All
   - [ ] Verify playback starts within 3 seconds
   - [ ] Verify navigation to Now Playing screen

3. **P3 - Play Individual Album**
   - [ ] From artist albums screen, tap an album
   - [ ] Verify album playback starts
   - [ ] Verify navigation to Now Playing screen

4. **Edge Cases**
   - [ ] Test artist with 0 albums (should show "No albums")
   - [ ] Test artist with 50+ albums (pagination)
   - [ ] Test with WiFi disconnected (offline state)
   - [ ] Monitor heap for 15 minutes of navigation

### Constitution Gates (§11)

- [ ] Cold boot <10 seconds
- [ ] Touch responds <100ms
- [ ] No heap decrease over 15 minutes
- [ ] No crashes during testing
