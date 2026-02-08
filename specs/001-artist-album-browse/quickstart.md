# Quickstart: Artist Album Browse

**Feature**: 001-artist-album-browse
**Date**: 2026-02-07

## Overview

This feature adds hierarchical browsing: tap an artist to see their albums, with "Play All" functionality.

## Files to Modify

| File | Changes |
|------|---------|
| `esphome/packages/browse.yaml` | Add globals, new category handler, artist albums script |
| `esphome/ui/browse_unified.yaml` | Add Play All button, modify tap handlers |
| `esphome/packages/navigation.yaml` | Handle back navigation from artist albums |

## Implementation Order

### Step 1: Add Globals (browse.yaml)

Add after existing browse globals (~line 150):

```yaml
# Artist album context
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

### Step 2: Add Category 6 Handler (browse.yaml)

In `browse_load_page` script, add case for category 6:

```cpp
} else if (id(current_browse_category) == 6) {
  // Artist Albums - filter by artist
  root["media_type"] = "album";
  root["artist"] = id(browse_context_artist_id).c_str();
  root["limit"] = 4;
  root["offset"] = id(browse_page_offset);
  root["order_by"] = "name";
}
```

### Step 3: Add browse_artist_albums Script (browse.yaml)

```yaml
- id: browse_artist_albums
  mode: single
  parameters:
    artist_id: string
    artist_name: string
  then:
    - lambda: |-
        // Save current position
        id(browse_saved_offset) = id(browse_page_offset);

        // Set artist context
        id(browse_context_artist_id) = artist_id;
        id(browse_context_artist_name) = artist_name;

        // Switch to artist albums mode
        id(current_browse_category) = 6;
        id(browse_page_offset) = 0;

        ESP_LOGI("browse", "Browsing albums for: %s", artist_name.c_str());
    - script.execute: browse_load_page
```

### Step 4: Add play_artist_all Script (browse.yaml)

```yaml
- id: play_artist_all
  mode: single
  then:
    - lambda: |-
        if (id(browse_context_artist_id).empty()) {
          ESP_LOGW("browse", "No artist selected for Play All");
          return;
        }
        ESP_LOGI("browse", "Playing all by: %s", id(browse_context_artist_name).c_str());
    - homeassistant.service:
        service: music_assistant.play_media
        data:
          entity_id: !lambda 'return id(selected_player_id).c_str();'
          media_id: !lambda 'return id(browse_context_artist_id).c_str();'
    - script.execute:
        id: navigate_to
        screen: 1  # Now Playing
```

### Step 5: Modify Artist Tap Handler (browse_unified.yaml)

Change item tap handlers for artists to navigate instead of play:

```yaml
on_click:
  - lambda: |-
      if (id(current_browse_category) == 5) {
        // Artists - navigate to albums
        auto aid = id(browse_item_0_media_id);
        auto aname = id(browse_item_0_title);
        // Script call handled below
      }
  - if:
      condition:
        lambda: 'return id(current_browse_category) == 5;'
      then:
        - script.execute:
            id: browse_artist_albums
            artist_id: !lambda 'return id(browse_item_0_media_id);'
            artist_name: !lambda 'return id(browse_item_0_title);'
      else:
        - script.execute:
            id: play_media_item
            media_id: !lambda 'return id(browse_item_0_media_id);'
            item_title: !lambda 'return id(browse_item_0_title);'
```

### Step 6: Add Play All Button (browse_unified.yaml)

Add in header area, visible only for category 6:

```yaml
- button:
    id: btn_browse_play_all
    x: 250
    y: 6
    width: 62
    height: 32
    hidden: true  # Shown dynamically when category == 6
    bg_color: 0x00D1FF
    on_click:
      - script.execute: play_artist_all
    widgets:
      - label:
          text: "\U000F040A"  # mdi:play
          text_font: icons_16
          text_color: 0x000000
          align: CENTER
```

### Step 7: Update Header Display Logic

When entering category 6, show artist name and Play All button:

```cpp
if (id(current_browse_category) == 6) {
  lv_label_set_text(id(lbl_browse_title), id(browse_context_artist_name).c_str());
  lv_obj_clear_flag(id(btn_browse_play_all), LV_OBJ_FLAG_HIDDEN);
} else {
  lv_obj_add_flag(id(btn_browse_play_all), LV_OBJ_FLAG_HIDDEN);
}
```

### Step 8: Handle Back Navigation (navigation.yaml)

In `navigate_back` script, add handling for category 6:

```cpp
if (id(current_browse_category) == 6) {
  // Return to Artists list
  id(current_browse_category) = 5;
  id(browse_page_offset) = id(browse_saved_offset);
  // Reload artists at saved position
  id(browse_artist_albums_returning) = true;  // Flag to skip animation
}
```

## Testing

1. Navigate to Artists screen
2. Tap an artist → should show their albums
3. Verify artist name in header
4. Verify Play All button visible
5. Tap Play All → should play and go to Now Playing
6. Navigate back → should return to Artists at same position
7. Tap an album → should play that album

## Common Issues

**Album list empty**: Check `artist` parameter format in API call
**Play All fails**: Verify selected player is valid
**Back navigation wrong**: Ensure `browse_saved_offset` is saved before category change
