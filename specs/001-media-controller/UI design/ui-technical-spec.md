# Technical Specification: Music Assistant DIY Controller

**Target Hardware**: 2.8" TFT LCD (320x240 Landscape)
**UI Framework**: LVGL via ESPHome
**Design Theme**: Music Assistant (Dark Mode, Deep Charcoal #121212, Rounded Corners, Vibrant Accents)

---

## 1. Global Navigation Flow

```
Boot/Standby: Lock Screen with Clock & Weather
       │
       ▼
     Home: Main Dashboard (4-tile grid)
       │
       ├──► Now Playing (Playback Control)
       │
       ├──► Library ──► Search (with Keyboard)
       │
       ├──► Players (Speaker Management) ──► Grouping Interface
       │
       ├──► Favorites (Synced from MA)
       │
       └──► System Settings
```

---

## 2. Detailed Screen Specifications

### Screen A: Lock Screen (Standby)

**Layout**: Minimalist, centered.

**Widgets**:
- **Digital Clock**: Large bold font (e.g., 48px)
- **Weather Widget**: Top-right. Icon (SVG/Font) + Temp label
- **Now Playing Footer**: Bottom 40px. Marquee text for "Artist - Title"

**Interaction**: Tap anywhere to navigate to Screen B (Home).

---

### Screen B: Home Dashboard

**Layout**: 2x2 Grid

**Tiles**:

| Tile | Content |
|------|---------|
| Now Playing | Mini-thumbnail + "Current Song" |
| Library | Icon + "Browse Music" |
| Speakers | Icon + "3 Active" (Dynamic status) |
| Favorites | Icon + "Your Stars" |

**Styles**: Tiles have 10px rounded corners and a slight border highlight on press.

---

### Screen C: Now Playing (Master Control)

**Layout**: Split Horizontal (Left 40% / Right 60%)

**Left Panel**:
- Album Art (rounded 12px) with subtle shadow

**Right Panel**:
- **Labels**: Track Title (Scroll/Marquee if long), Artist, Album
- **Controls**: Horizontal row `[Prev]` `[Play/Pause]` `[Next]` (Icons only, 40px size)
- **Output Button**: Small "Cast" icon button that links to Screen D

**Bottom**:
- Full-width Progress Bar (2px height)
- Volume Slider (below progress bar)

---

### Screen D: Player Management (MA Style)

**Layout**: Vertical scrollable list

**Row Content**:
- Icon (Speaker type)
- Label (Zone Name)
- Toggle Switch (Power)
- Horizontal Slider (Volume) taking up 50% of row width

**Header**: "Group" button to enter Screen E

---

### Screen E: Multi-Room Grouping

**Layout**: Checklist

**Items**: Speaker names with Checkboxes

**Action**: Large "Apply" button at bottom

---

### Screen F: Global Search & Keyboard

**Layout**: Top-aligned Text Area + Bottom-aligned Keyboard

**Keyboard**: QWERTY layout. Key size approx 30x30px for touch accuracy.

**Results**: Overlay or dropdown list above the keyboard

---

## 3. Technical Implementation Notes

### Touch Targets

All buttons MUST be at least **40x40px** to accommodate finger touches on a 2.8" screen.

### Animations

Use fast transitions (**200ms**) for screen sliding to keep the DIY hardware feeling responsive.

### Assets

Use **FontAwesome** or **Material Design Icons** for:
- Play, Pause, Skip
- Speaker icons
- Search icon
- Navigation icons

### Visual States

| State | Style |
|-------|-------|
| Active speakers | Bright accent color (e.g., #00D1FF) |
| Inactive items | Semi-transparent or greyed out |
| Pressed/Touch | Slight border highlight or scale |

---

## 4. Event Handling & Logic

### 4.1 Navigation Logic (Screen Switching)

Use a "Screen Manager" approach—do not delete screens, navigate between screen pointers.

**Forward Navigation**:
```
lv_scr_load_anim(new_screen, LV_SCR_LOAD_ANIM_MOVE_LEFT, 300, 0, false)
```

**Back Navigation**:
Every sub-screen (Library, Speakers, Search) needs a "Back" button that loads the Home screen with `ANIM_MOVE_RIGHT` transition for natural spatial flow.

---

### 4.2 Now Playing Update Logic

Since music metadata (Title, Artist, Progress) changes constantly, the UI should NOT refresh the whole screen.

**Implementation**:
- Create an Update Task using `lv_timer_create` that runs every **500ms**
- Update only the `lv_label` text for track title
- Update only the `lv_slider` value for progress bar
- For long track titles, set label long mode to `LV_LABEL_LONG_SCROLL_CIRCULAR`

---

### 4.3 Volume & Slider Synchronization

To avoid "jumping" sliders when touched:

**Event**: Use `LV_EVENT_VALUE_CHANGED` for visual feedback

**API Call**: Send new volume value to Music Assistant API only on `LV_EVENT_RELEASED`

**Rationale**: Prevents flooding Wi-Fi/Server with dozens of requests per second while sliding

---

### 4.4 Speaker Grouping Multi-Select

**State Storage**: Store checkbox states in a local bitmask or array

**Apply Action**:
1. Event handler loops through checklist
2. Identifies which speakers are checked
3. Sends a single `massistant.group_players()` command to server

---

### 4.5 Keyboard Integration

**Linking**: The `lv_keyboard` must be linked to the `lv_textarea`

**OK Button Action**:
1. Trigger API search call
2. Hide keyboard
3. Show results list

---

## 5. Screen Asset Reference

| Screen | Folder | Files |
|--------|--------|-------|
| Home Dashboard | `stitch/home_dashboard_(entry)/` | `screen.png`, `code.html` |
| Now Playing | `stitch/now_playing_(control)/` | `screen.png`, `code.html` |
| Player & Zone Management | `stitch/player_&_zone_management/` | `screen.png`, `code.html` |
| Music Library Hub | `stitch/music_library_hub/` | `screen.png`, `code.html` |
| Sync'd Favorites Grid | `stitch/sync'd_favorites_grid/` | `screen.png`, `code.html` |
| Search & Keyboard UI | `stitch/search_&_keyboard_ui/` | `screen.png`, `code.html` |
| Active Play Queue | `stitch/active_play_queue/` | `screen.png`, `code.html` |
| Multi-room Grouping UI | `stitch/multi-room_grouping_ui/` | `screen.png`, `code.html` |
| Device Settings & Status | `stitch/device_settings_&_status/` | `screen.png`, `code.html` |

---

## 6. Design Constants

```
/* Colors */
BACKGROUND_PRIMARY:    #121212  /* Deep Charcoal */
BACKGROUND_SECONDARY:  #1E1E1E  /* Slightly lighter */
ACCENT_PRIMARY:        #00D1FF  /* Vibrant Cyan */
ACCENT_SECONDARY:      #FF6B35  /* Orange accent */
TEXT_PRIMARY:          #FFFFFF  /* White */
TEXT_SECONDARY:        #B3B3B3  /* Grey */
INACTIVE:              #666666  /* Greyed out */

/* Dimensions */
SCREEN_WIDTH:          320px
SCREEN_HEIGHT:         240px
MIN_TOUCH_TARGET:      40px
CORNER_RADIUS_SMALL:   6px
CORNER_RADIUS_MEDIUM:  10px
CORNER_RADIUS_LARGE:   12px

/* Timing */
TRANSITION_FAST:       200ms
TRANSITION_NORMAL:     300ms
UPDATE_INTERVAL:       500ms
```

---

## 7. Constitution Compliance Notes

Per the project constitution (§6, §9):

- **Touch targets**: Minimum 44x44px (constitution) vs 40x40px (this spec) — **use 44px**
- **Touch feedback**: Must respond within 100ms (constitution aligns with 200ms transitions)
- **Progress updates**: Rate-limited to 2-4 Hz (constitution) aligns with 500ms timer
- **Album art**: Must display within 3 seconds of track change (per updated spec)
- **No UI stutter**: All animations and loading must not block main loop
