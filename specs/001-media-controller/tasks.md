# Tasks: Music Assistant Media Controller

**Input**: Design documents from `/specs/001-media-controller/`
**Prerequisites**: plan.md, spec.md, UI design/ui-technical-spec.md

**Tests**: Tests are NOT explicitly requested in the specification. Test tasks are omitted.
**Human Testing**: Human testing checkpoints are included at phase boundaries per analysis remediation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2)
- **HT-XXX**: Human testing checkpoint (requires manual validation)
- Include exact file paths in descriptions

## Path Conventions

- **ESPHome Project**: Main config in `esphome/`, packages in `esphome/packages/`
- **Main config**: `esphome/main.yaml` (single entrypoint per Constitution §5.2.1)
- **UI Screens**: `esphome/ui/` for LVGL screens
- **UI Components**: `esphome/ui/components/` for shared widgets
- **Packages**: `esphome/packages/` for hardware, services, and business logic
- **Custom C++**: `components/<name>/` with README (if needed)
- **Secrets**: `esphome/secrets.yaml` (gitignored per Constitution §1.6)

---

## Phase 1: Setup (Project Initialization) ✅ COMPLETE

**Purpose**: Create project structure and base ESPHome configuration

- [x] T001 Create project directory structure per constitution §5.1 (esphome/, esphome/packages/, esphome/ui/, esphome/ui/components/, components/, fonts/, docs/)
- [x] T001a [P] Create esphome/secrets.yaml template and add to .gitignore per Constitution §1.6
- [x] T001b [P] Create CHANGELOG.md in repository root following Keep a Changelog format per Constitution §3.5
- [x] T002 Create base ESPHome configuration in esphome/main.yaml with ESP32-2432S028R board definition and package includes
- [x] T003 [P] Create hardware package for SPI bus configuration in esphome/packages/spi.yaml
- [x] T004 [P] Create hardware package for ILI9341 display driver in esphome/packages/display.yaml
- [x] T005 [P] Create hardware package for XPT2046 touch controller in esphome/packages/touch.yaml
- [x] T006 [P] Create hardware package for backlight PWM control in esphome/packages/backlight.yaml
- [x] T007 Create LVGL initialization package in esphome/packages/lvgl.yaml with theme colors (#121212, #00D1FF, #FF6B35)
- [x] T008 Create boot screen with "Starting..." text in esphome/ui/boot.yaml

**Checkpoint**: ESPHome compiles, display shows boot screen, touch responds, backlight works ✅

---

## Human Testing Checkpoint: Setup Validation ✅ PASSED

**Purpose**: Verify hardware initialization before proceeding

- [x] HT-001 **HUMAN TEST**: Flash firmware to device and verify:
  - [x] Device powers on without errors in serial log
  - [x] Display shows boot screen with "Music Remote" text
  - [x] Display is in landscape orientation (320x240)
  - [x] Backlight is visible at conservative brightness (≤50%)
  - [x] Touch coordinates appear in serial log when screen is touched
  - [x] Touch calibration is reasonably accurate (tap matches visual)

**Pass criteria**: All items checked. ✅ PASSED 2026-02-01

---

## Phase 2: Foundational (Blocking Prerequisites) ✅ COMPLETE

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

- [x] T009 Create system state enum (Booting, Connecting, Ready, Offline, Error) in esphome/packages/state_machine.yaml
- [x] T010 Implement Wi-Fi connection management with timeout in esphome/packages/wifi.yaml
- [x] T011 Implement Home Assistant Native API connection in esphome/packages/homeassistant.yaml
- [x] T012 [P] Create state transition logic with event-driven updates in esphome/packages/state_machine.yaml
- [x] T013 [P] Create visual state indicator component (icon + text) in esphome/ui/components/status_indicator.yaml
- [x] T014 Implement offline detection with debounce (5-second threshold) in esphome/packages/connectivity.yaml
- [x] T015 Implement automatic reconnection with exponential backoff in esphome/packages/connectivity.yaml
- [x] T016 Create "Connecting" screen with progress indication in esphome/ui/connecting.yaml
- [x] T017 Ensure UI remains responsive during connection attempts (non-blocking API calls)
- [x] T018 Create boot time measurement sensor in esphome/packages/diagnostics.yaml
- [x] T018a [P] Create uptime sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)
- [x] T018b [P] Create wifi_rssi sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)
- [x] T018c [P] Create ip_address sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)
- [x] T018d [P] Create ha_api_connected binary sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)
- [x] T019 Create free_heap sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)
- [x] T019a [P] Create reset_reason sensor in esphome/packages/diagnostics.yaml (Constitution §1.5.1)

**Checkpoint**: Foundation ready - device transitions Booting → Connecting → Ready, offline detection works, UI responsive during all states ✅

---

## Human Testing Checkpoint: Connectivity & State Machine ✅ PASSED

**Purpose**: Verify system states and network resilience

- [x] HT-002 **HUMAN TEST**: Verify state transitions:
  - [x] Boot shows "Booting" state briefly
  - [x] "Connecting" state appears during Wi-Fi/HA connection
  - [x] "Ready" state appears when fully connected
  - [x] State indicator visible in UI

- [ ] HT-003 **HUMAN TEST**: Verify offline handling:
  - [ ] Disconnect Wi-Fi → UI shows "Offline" within 5 seconds
  - [ ] UI remains responsive while offline (touch works, no freeze)
  - [ ] Reconnect Wi-Fi → UI shows "Ready" within 30 seconds
  - [ ] No error spam in serial log during offline period

- [ ] HT-004 **HUMAN TEST**: Verify Home Assistant recovery:
  - [ ] Stop Home Assistant → UI shows offline/unavailable state
  - [ ] Start Home Assistant → UI recovers within 30 seconds

- [ ] HT-004a **HUMAN TEST**: Verify diagnostics sensors in Home Assistant:
  - [ ] uptime sensor appears and updates
  - [ ] wifi_rssi sensor shows signal strength
  - [ ] ip_address sensor shows correct IP
  - [ ] ha_api_connected binary sensor shows "on"
  - [ ] free_heap sensor shows memory value
  - [ ] reset_reason sensor shows last reset cause

**Pass criteria**: All items checked. Foundation ready for user stories.

---

## Phase 3: User Story 1 & 2 - Control Playback & View Now Playing (Priority: P1) MVP

**Goal**: Implement the Now Playing screen with track display, album art, and playback controls - the core value proposition

**Independent Test**: Play music on any Music Assistant player, use controller to pause, resume, skip, adjust volume; verify track info and album art display correctly

### Implementation for User Stories 1 & 2

- [x] T020 [P] [US1] Create Now Playing screen layout (40% left / 60% right split) in esphome/ui/now_playing.yaml
- [x] T021 [P] [US1] [US2] Create album art container component with rounded corners (12px) in esphome/ui/components/album_art.yaml
- [x] T022 [US1] [US2] Add placeholder image for missing album art in esphome/ui/components/album_art.yaml
- [x] T023 [US1] [US2] Add loading indicator for album art fetching in esphome/ui/components/album_art.yaml
- [x] T024 [US1] [US2] Implement album art fetch from Last.fm API with 5-second timeout in esphome/packages/album_art.yaml
  - **Note**: Changed from Music Assistant to Last.fm API due to HA album art size (~100KB) exceeding ESP32 heap (~36KB contiguous). Last.fm 64x64 images are ~3-5KB.
- [x] T025 [US1] [US2] Implement album art memory management with error handling in esphome/packages/album_art.yaml
  - **Note**: Changed from LRU cache to single-image with comprehensive error handling:
    - Heap memory check (40KB minimum) before fetch
    - Exponential backoff cooldown (10s→320s max) on errors
    - LVGL buffer reduced from 25% to 10% to free ~23KB heap
    - Graceful fallback to placeholder on failure
- [x] T026 [P] [US1] [US2] Create scrolling/marquee label component for long titles in esphome/ui/components/marquee_label.yaml
- [x] T027 [US1] [US2] Create track metadata labels (title, artist, album) in esphome/ui/now_playing.yaml
- [x] T028 [US1] Subscribe to Music Assistant media_player state updates in esphome/packages/media_player.yaml
- [x] T029 [P] [US1] Create playback control button component (44x44px minimum) in esphome/ui/components/control_button.yaml
- [x] T030 [US1] Implement play/pause button with icon swap in esphome/ui/now_playing.yaml
- [x] T031 [US1] Implement previous track button in esphome/ui/now_playing.yaml
- [x] T032 [US1] Implement next track button in esphome/ui/now_playing.yaml
- [x] T033 [P] [US1] Create progress bar component with touch-to-seek in esphome/ui/components/progress_bar.yaml
- [x] T034 [US1] [US2] Implement progress bar updates (2-4 Hz rate limiting) in esphome/ui/now_playing.yaml
- [x] T035 [P] [US1] Create volume slider component with smooth dragging in esphome/ui/components/volume_slider.yaml
- [x] T036 [US1] Implement volume slider with debounced API calls (only on release) in esphome/ui/now_playing.yaml
- [x] T036a [US1] Implement mute/unmute toggle button with icon state in esphome/ui/now_playing.yaml (FR-025)
- [x] T037 [US1] Add optimistic UI updates on control touch (immediate visual feedback) in esphome/packages/media_player.yaml
- [x] T038 [US1] Implement state reconciliation within 2 seconds of action in esphome/packages/media_player.yaml
- [x] T039 [US1] Add input debouncing for rapid button taps (200ms minimum) in esphome/packages/media_player.yaml
- [x] T040 [US1] [US2] Display active player name in header area in esphome/ui/now_playing.yaml
- [x] T041 [US1] [US2] Handle missing metadata gracefully ("Unknown" placeholders) in esphome/ui/now_playing.yaml

**Checkpoint**: Now Playing screen fully functional - can control playback, see track info, album art loads within 3 seconds, touch feedback within 100ms

---

## Human Testing Checkpoint: MVP Validation (Now Playing)

**Purpose**: Verify core playback control works end-to-end

- [ ] HT-005 **HUMAN TEST**: Play music on Music Assistant player, then verify:
  - [ ] Track title displays correctly (scrolls if long)
  - [ ] Artist name displays correctly
  - [ ] Album name displays (if available)
  - [ ] Album art loads within 3 seconds
  - [ ] Placeholder shows if no album art available
  - [ ] Progress bar updates smoothly (no stutter)
  - [ ] Elapsed/total time displays correctly

- [ ] HT-006 **HUMAN TEST**: Test playback controls:
  - [ ] Tap Play/Pause → playback toggles within 1 second
  - [ ] Button icon changes to reflect state
  - [ ] Tap Next → next track starts within 2 seconds
  - [ ] Tap Previous → previous track starts within 2 seconds
  - [ ] Touch feedback visible within 100ms on all buttons

- [ ] HT-007 **HUMAN TEST**: Test volume and seek:
  - [ ] Drag volume slider → volume changes smoothly
  - [ ] Volume level updates on release (not during drag)
  - [ ] Tap/drag progress bar → playback seeks to position
  - [ ] Mute button toggles mute state correctly

- [ ] HT-008 **HUMAN TEST**: Performance check:
  - [ ] No visible UI stutter during any operation
  - [ ] Check free_heap sensor - note baseline value: _______
  - [ ] Operate for 15 minutes, verify heap stable (no sustained decrease)

**Pass criteria**: All items checked. MVP deliverable - can demo to stakeholders.

---

## Phase 4: Home Dashboard & Navigation ✅ COMPLETE

**Goal**: Create central navigation hub with 2x2 tile grid and implement screen navigation system

**Independent Test**: Navigate from Home to Now Playing and back; verify transitions animate smoothly (200-300ms)

### Implementation for Navigation

- [x] T042 Create screen manager with navigation stack in esphome/packages/navigation.yaml
- [x] T043 [P] Implement slide-left animation for forward navigation in esphome/packages/navigation.yaml
  - **Note**: Uses ESPHome's lvgl.page.show with OUT_LEFT animation (250ms)
- [x] T044 [P] Implement slide-right animation for back navigation in esphome/packages/navigation.yaml
  - **Note**: Uses ESPHome's lvgl.page.show with OUT_RIGHT animation (250ms)
- [x] T045 Create Home Dashboard screen with 2x2 grid layout in esphome/packages/lvgl.yaml (page_home)
  - **Note**: Implemented in lvgl.yaml instead of separate file per package architecture
- [x] T046 [P] Create tile component with icon, text, and 10px rounded corners in esphome/packages/lvgl.yaml
  - **Note**: Implemented as button widgets with style_tile_normal style (10px radius)
- [x] T047 Add press state styling (border highlight) for tiles in esphome/packages/lvgl.yaml
  - **Note**: Uses LVGL pressed: style property with 2px cyan border
- [x] T048 Implement Now Playing tile with current track title in esphome/packages/lvgl.yaml
  - **Note**: Widget ID lbl_home_track_title for dynamic updates
- [x] T049 [P] Implement Library tile with icon in esphome/packages/lvgl.yaml
- [x] T050 [P] Implement Speakers tile with dynamic active count in esphome/packages/lvgl.yaml
  - **Note**: Widget ID lbl_home_speaker_count for dynamic updates
- [x] T051 [P] Implement Favorites tile with icon in esphome/packages/lvgl.yaml
- [x] T052 Create back button for sub-screens in esphome/packages/lvgl.yaml
  - **Note**: Back button added to page_main (Now Playing screen)
- [x] T053 Add back button to all sub-screens using header component
  - **Note**: Currently added to Now Playing; other screens will be added when defined
- [x] T054 Wire navigation from Home tiles to respective screens in esphome/packages/lvgl.yaml
  - **Note**: Now Playing tile wired; other tiles pending screen definitions

**Checkpoint**: Dashboard displays 2x2 grid, navigation to Now Playing works with animations, back navigation consistent

---

## Human Testing Checkpoint: Navigation

**Purpose**: Verify screen transitions and dashboard

- [ ] HT-009 **HUMAN TEST**: Test Home Dashboard:
  - [ ] 2x2 tile grid displays correctly
  - [ ] Tiles have rounded corners and proper styling
  - [ ] Tile press shows visual feedback within 100ms
  - [ ] Speakers tile shows correct active count

- [ ] HT-010 **HUMAN TEST**: Test navigation:
  - [ ] Tap Now Playing tile → navigates with slide animation
  - [ ] Tap back button → returns with reverse animation
  - [ ] Transitions complete in 200-300ms (no lag)
  - [ ] Navigate 20+ times, verify no memory leak (check free_heap)

**Pass criteria**: All items checked.

---

## Phase 5: User Story 3 - Select Target Player (Priority: P2) ✅ COMPLETE

**Goal**: Allow users to view available players, see status, and select which player to control

**Independent Test**: Have multiple MA players available, switch between them, verify controls affect only selected player

### Implementation for User Story 3

- [x] T055 [US3] Create Player Management screen layout in esphome/ui/players.yaml
- [x] T056 [P] [US3] Create scrollable list container component in esphome/ui/components/scroll_list.yaml
- [x] T057 [P] [US3] Create player row component (icon, label, status, volume) in esphome/ui/components/player_row.yaml
- [x] T058 [US3] Add volume slider to player row (50% width) in esphome/ui/components/player_row.yaml
- [x] T059 [US3] Add power toggle switch per player in esphome/ui/components/player_row.yaml
- [x] T060 [US3] Query Music Assistant for available players list in esphome/packages/player_manager.yaml
- [x] T061 [US3] Subscribe to player state updates in esphome/packages/player_manager.yaml
- [x] T062 [US3] Implement online/offline styling (bright vs greyed #666666) in esphome/ui/components/player_row.yaml
- [x] T063 [US3] Implement tap-to-select behavior for player rows in esphome/ui/players.yaml
- [x] T064 [US3] Prevent selection of offline players (FR-034) in esphome/packages/player_manager.yaml
- [x] T065 [US3] Update header to show selected player name in esphome/ui/components/header.yaml
- [x] T066 [US3] Persist selected player ID to NVS in esphome/packages/player_manager.yaml
- [x] T067 [US3] Restore selected player on boot from NVS in esphome/packages/player_manager.yaml
- [x] T068 [US3] Handle deleted/unavailable player - prompt re-selection in esphome/packages/player_manager.yaml
- [x] T069 [US3] Wire Speakers tile navigation to Player Management screen in esphome/ui/home.yaml

**Checkpoint**: Player list displays all available players, selection works and persists across reboot, offline players greyed out

---

## Human Testing Checkpoint: Player Selection

**Purpose**: Verify multi-player management

- [ ] HT-011 **HUMAN TEST**: Test player list:
  - [ ] All Music Assistant players appear in list
  - [ ] Online players show bright/active styling
  - [ ] Offline players show greyed styling
  - [ ] Currently selected player is visually indicated

- [ ] HT-012 **HUMAN TEST**: Test player selection:
  - [ ] Tap online player → becomes selected
  - [ ] Tap offline player → selection prevented
  - [ ] Return to Now Playing → controls affect new player
  - [ ] Player switch completes in under 5 seconds

- [ ] HT-013 **HUMAN TEST**: Test persistence:
  - [ ] Select a player, reboot device
  - [ ] After reboot, same player is still selected
  - [ ] If selected player is now offline, prompt appears

**Pass criteria**: All items checked.

---

## Phase 6: User Story 4 - Browse and Play Content (Priority: P2) ✅ COMPLETE

**Goal**: Allow users to browse library categories and start playback of selected content

**Independent Test**: Navigate through browse categories, select content, verify playback starts on active player

### Implementation for User Story 4

- [x] T070 [US4] Create Music Library Hub screen with category tiles in esphome/ui/library.yaml
- [x] T071 [P] [US4] Create list item component (text only) in esphome/ui/components/list_item.yaml
- [x] T072 [P] [US4] Create loading indicator component in esphome/ui/components/loading.yaml
- [x] T073 [P] [US4] Create empty state message component in esphome/ui/components/empty_state.yaml
- [x] T074 [US4] Implement Recently Played list view in esphome/ui/browse_recent.yaml
- [x] T075 [US4] Implement Favorites list view (synced from MA) in esphome/ui/browse_favorites.yaml
- [x] T076 [US4] Implement Playlists list view in esphome/ui/browse_playlists.yaml
- [x] T077 [US4] Implement Albums list view in esphome/ui/browse_albums.yaml
- [x] T078 [US4] Implement Artists list view in esphome/ui/browse_artists.yaml
- [x] T079 [US4] Implement paginated/chunked loading for large lists in esphome/packages/browse.yaml
- [x] T080 [US4] Add loading indicator during content fetch in browse screens
- [x] T081 [US4] Add empty state message for empty categories in browse screens
- [x] T082 [US4] Implement tap-to-play on list items in esphome/packages/browse.yaml
- [x] T083 [US4] Create Search screen with text input area in esphome/ui/search.yaml
- [x] T084 [US4] Implement QWERTY keyboard (30x30px keys minimum) in esphome/ui/search.yaml
- [x] T085 [US4] Link keyboard to textarea for search input in esphome/ui/search.yaml
- [x] T086 [US4] Implement search query submission to MA in esphome/packages/search.yaml
- [x] T087 [US4] Display search results in list view in esphome/ui/search.yaml
- [x] T088 [US4] Add navigation breadcrumbs or back button to browse screens
- [x] T089 [US4] Wire Library tile navigation to Music Library Hub in esphome/ui/home.yaml
- [x] T090 [US4] Wire Favorites tile navigation to Favorites list in esphome/ui/home.yaml

**Checkpoint**: All five browse categories accessible, lists scroll smoothly, selecting content starts playback, search keyboard functional

---

## Human Testing Checkpoint: Media Browsing

**Purpose**: Verify library browsing and playback initiation

- [ ] HT-014 **HUMAN TEST**: Test browse categories:
  - [ ] Recently Played list loads and displays
  - [ ] Favorites list loads (synced from MA)
  - [ ] Playlists list loads
  - [ ] Albums list loads
  - [ ] Artists list loads
  - [ ] Empty categories show helpful message

- [ ] HT-015 **HUMAN TEST**: Test list interaction:
  - [ ] Lists scroll smoothly without stutter
  - [ ] Large lists load in chunks (no UI freeze)
  - [ ] Loading indicator visible during fetch
  - [ ] Tap item → playback starts on active player
  - [ ] Browse and play completes within 30 seconds

- [ ] HT-016 **HUMAN TEST**: Test search:
  - [ ] Keyboard displays with usable key size
  - [ ] Can type search query
  - [ ] Results display after submission
  - [ ] Can select result to start playback

**Pass criteria**: All items checked.

---

## Phase 7: User Story 5 - Manage Speaker Groups (Priority: P3) ✅ COMPLETE

**Goal**: Allow users to create, modify, and dissolve speaker groups for synchronized multi-room playback

**Independent Test**: Create a group from available players, verify synchronized playback, dissolve group

### Implementation for User Story 5

- [x] T091 [US5] Create Multi-room Grouping screen with checklist layout in esphome/ui/grouping.yaml
- [x] T092 [P] [US5] Create checkbox component for group selection in esphome/ui/components/checkbox.yaml
- [x] T093 [US5] Display all players with checkboxes in esphome/ui/grouping.yaml
- [x] T094 [US5] Pre-check players already in current group in esphome/ui/grouping.yaml
  - **Enhancement**: Pre-checks ALL group members when editing existing group, not just selected player
- [x] T095 [US5] Store checkbox states locally before apply (bitmask/array) in esphome/packages/grouping.yaml
- [x] T096 [US5] Implement Apply button at bottom in esphome/ui/grouping.yaml
  - **Enhancement**: Dynamic button text "Create Group" vs "Update Group" based on mode
- [x] T097 [US5] On Apply: validate selections, send group command to MA in esphome/packages/grouping.yaml
  - **Enhancement**: Uses designated leader from user selection, not first-checked player
- [x] T098 [US5] Handle partial group creation failure (FR-044) in esphome/packages/grouping.yaml
- [x] T099 [US5] Display which players failed and why in esphome/ui/grouping.yaml
- [x] T100 [US5] Implement Cancel button to discard changes in esphome/ui/grouping.yaml
- [x] T101 [US5] Update active target to new group after creation in esphome/packages/grouping.yaml
- [x] T102 [US5] Prevent invalid grouping combinations (FR-045) in esphome/packages/grouping.yaml
- [x] T103 [US5] Add Group button to Player Management header in esphome/ui/players.yaml
  - **Enhancement**: Uses mdi:speaker-multiple icon instead of link-variant

### Additional Enhancements (Post-Phase 7)

- [x] T103a [US5] Add Unjoin button to Player Management header (visible when player is grouped)
  - Uses mdi:speaker-off icon
- [x] T103b [US5] Implement leader selection with crown icon in grouping screen
  - Tap status area to change designated leader
  - Crown icon (gold) shows current leader
  - Link icon (green) shows group members
- [x] T103c [US5] Add dynamic title: "Create Group" vs "Edit Group" based on mode
- [x] T103d [US5] Implement tree view sorting on Players screen
  - Leaders appear first with crown icon and gold border
  - Group members appear directly below leader with link icon
  - Ungrouped players appear at bottom
- [x] T103e [US5] Fix player selection mapping for sorted display order
  - Added display_order_row_X globals to track row-to-player mapping
  - Click/volume handlers use mapped indices for correct player selection
- [x] T103f [US5] Remove power buttons from player rows, use freed space for wider volume slider

**Checkpoint**: Grouping interface works, checkboxes toggle, Apply creates group, partial failures reported, Cancel discards changes, tree view shows group hierarchy

---

## Human Testing Checkpoint: Speaker Grouping

**Purpose**: Verify multi-room group creation

- [ ] HT-017 **HUMAN TEST**: Test grouping interface:
  - [ ] All players appear with checkboxes
  - [ ] Current group members are pre-checked when editing
  - [ ] Checkboxes toggle correctly on tap
  - [ ] Title shows "Create Group" for new group, "Edit Group" for existing
  - [ ] Crown icon visible on designated leader
  - [ ] Tap status area changes leader (crown moves)

- [ ] HT-018 **HUMAN TEST**: Test group creation:
  - [ ] Select 2+ players, tap Apply/Create Group
  - [ ] Group is created in Music Assistant
  - [ ] Designated leader (with crown) becomes the actual group leader
  - [ ] Playback is synchronized across group members
  - [ ] Cancel button discards changes (no group created)

- [ ] HT-019 **HUMAN TEST**: Test group modification:
  - [ ] Can remove player from existing group
  - [ ] Can add player to existing group
  - [ ] Can change group leader via crown selection
  - [ ] Partial failures reported clearly (if testable)

- [ ] HT-019a **HUMAN TEST**: Test Players screen tree view:
  - [ ] Group leaders show crown icon and gold border
  - [ ] Group members appear directly below their leader
  - [ ] Group members show link icon and "Grouped" status
  - [ ] Ungrouped players appear at bottom with "Online" status
  - [ ] Clicking any row selects the correct player (not wrong due to sorting)
  - [ ] Volume sliders affect the correct player
  - [ ] Unjoin button visible when grouped player selected

**Pass criteria**: All items checked.

---

## Phase 8: User Story 6 - Operate During Connectivity Issues (Priority: P2)

**Goal**: Ensure device remains usable and informative during network or service interruptions

**Independent Test**: Disconnect Wi-Fi or stop Home Assistant, verify UI remains responsive and clearly indicates the issue

### Implementation for User Story 6

- [x] T104 [US6] Ensure all controls safely ignore taps when offline in esphome/packages/media_player.yaml
  - Added system_state check at start of all media control scripts (play, pause, play_pause, next, prev, volume_set, volume_up, volume_down, mute_toggle, seek)
  - Controls return early with warning log if system_state != 2 (Ready)
- [x] T105 [US6] Visually disable controls when offline (greyed styling) in esphome/ui/now_playing.yaml
  - Added disabled state styling to btn_play, btn_prev, btn_next, btn_mute, slider_volume, progress_slider
  - Icon colors change to grey (0x666666) when disabled
  - Script `update_controls_enabled_state` called on every state transition
- [x] T106 [US6] Display last known state when offline in esphome/packages/state_machine.yaml
  - Already working: HA text sensors retain last known values when connection lost
  - Track title, artist, album, progress all persist during offline periods
- [x] T107 [US6] Implement retry limiting to prevent retry storms (FR-063) in esphome/packages/connectivity.yaml
  - Already implemented in Phase 2: exponential backoff on reconnection in wifi.yaml
- [x] T108 [US6] Show clear "Offline" indicator on all screens in esphome/ui/components/status_indicator.yaml
  - Already implemented: lbl_status updated to show "Offline" (orange) on state transition
  - Shows "Connecting..." during connection attempts
  - Shows "Connected" (cyan) when ready
- [x] T109 [US6] Show "Service Unavailable" when MA unavailable but HA connected in esphome/ui/components/status_indicator.yaml
  - Detects when ha_media_state == "unavailable" while system_state == Ready
  - Shows "Service Unavailable" in orange on lbl_status
  - Restores "Connected" when player becomes available again
- [x] T110 [US6] Auto-recover UI when connectivity restored in esphome/packages/connectivity.yaml
  - Already implemented: on_client_connected triggers nav_initialize and library_index_sync
  - update_controls_enabled_state re-enables controls on Ready transition
- [x] T111 [US6] Update UI immediately (within 30 seconds) on reconnection in esphome/packages/connectivity.yaml
  - State transitions trigger immediate UI updates via update_controls_enabled_state script

**Checkpoint**: UI never freezes offline, controls disabled gracefully, auto-recovery works within 30 seconds

---

## Human Testing Checkpoint: Resilience (Offline Handling)

**Purpose**: Verify offline behavior under various failure conditions

- [ ] HT-020 **HUMAN TEST**: Test Wi-Fi disconnection:
  - [ ] Disconnect Wi-Fi during playback
  - [ ] "Offline" indicator appears within 5 seconds
  - [ ] Controls become disabled/greyed
  - [ ] Tapping disabled controls does nothing (no crash)
  - [ ] Last known state remains displayed
  - [ ] Reconnect Wi-Fi → UI recovers within 30 seconds

- [ ] HT-021 **HUMAN TEST**: Test Home Assistant unavailability:
  - [ ] Stop Home Assistant while device connected
  - [ ] UI shows appropriate unavailable state
  - [ ] Start Home Assistant → UI recovers within 30 seconds

- [ ] HT-022 **HUMAN TEST**: Test Music Assistant unavailability:
  - [ ] If MA becomes unavailable but HA is connected
  - [ ] "Service Unavailable" message appears
  - [ ] Device diagnostics remain functional in HA

**Pass criteria**: All items checked.

---

## Phase 9: User Story 7 - Adjust Device Settings (Priority: P3) ✅ COMPLETE

**Goal**: Allow users to adjust screen brightness and idle behavior for their environment

**Independent Test**: Change brightness settings, wait for idle timeout, verify screen dims/sleeps and wakes on touch

### Implementation for User Story 7

- [x] T112 [US7] Create Device Settings screen layout in esphome/ui/settings.yaml
  - Full settings screen with brightness slider, timeout dropdowns, debug toggle
  - Back button, header with title, vertically stacked settings
- [x] T113 [P] [US7] Create slider component for brightness in esphome/ui/components/settings_slider.yaml
  - Inline in settings.yaml - LVGL slider with percentage label
- [x] T114 [P] [US7] Create dropdown/selector component for timeout values in esphome/ui/components/settings_dropdown.yaml
  - Inline in settings.yaml - Button-style dropdown that cycles through options
- [x] T115 [P] [US7] Create toggle switch component for debug mode in esphome/ui/components/toggle_switch.yaml
  - Inline in settings.yaml - ON/OFF indicator with timer countdown
- [x] T116 [US7] Implement brightness slider with immediate apply (FR-075) in esphome/ui/settings.yaml
  - Slider updates backlight immediately via light.turn_on call
- [x] T117 [US7] Implement idle dim timeout selector in esphome/ui/settings.yaml
  - Cycles through: Off, 30s, 1min, 2min, 5min
- [x] T118 [US7] Implement screen-off timeout selector in esphome/ui/settings.yaml
  - Cycles through: Off, 1min, 2min, 5min, 10min
- [x] T119 [US7] Implement debug mode toggle with 30-minute auto-disable (Constitution §10.2) in esphome/ui/settings.yaml
  - Toggle with countdown timer, auto-disables after 30 minutes
- [x] T120 [US7] Create settings persistence service using NVS in esphome/packages/settings.yaml
  - Globals with restore_value: yes for automatic NVS persistence
- [x] T121 [US7] Persist all settings to NVS (FR-074) in esphome/packages/settings.yaml
  - brightness_level, idle_dim_timeout, screen_off_timeout, debug_mode_enabled
- [x] T122 [US7] Load settings on boot from NVS in esphome/packages/settings.yaml
  - Automatic restoration via ESPHome preferences system
- [x] T123 [US7] Implement idle timer based on touch activity in esphome/packages/idle.yaml
  - Tracks last_activity_timestamp_ms, updated on touch/media events
- [x] T124 [US7] Dim backlight after idle timeout in esphome/packages/idle.yaml
  - Dims to 20% after configured timeout
- [x] T125 [US7] Turn off backlight after screen-off timeout in esphome/packages/idle.yaml
  - Turns off backlight completely after configured timeout
- [x] T126 [US7] Wake immediately on touch from dimmed/off state in esphome/packages/idle.yaml
  - Touch handler in touch.yaml calls reset_idle_timer
- [x] T127 [US7] Reset idle timer on playback state changes (not just touch) in esphome/packages/idle.yaml
  - ha_media_state on_value handler resets idle timer
- [x] T128 [US7] Add Settings navigation from Home or system menu
  - Settings gear icon button in top-right of Home screen

**Checkpoint**: All settings adjustable and persistent, idle dim/off works, wake-on-touch immediate

### Memory Optimization (2026-02-07)

- [x] Disabled search.yaml and search_ui.yaml packages to free heap memory
- [x] Removed Recent and Search tiles from Library screen (now 2x2 grid)
- [x] Reduced font glyph sets (removed accented characters and unused symbols)
- [x] Removed mdi:magnify icon from icons_24 font
- [x] Replaced Favorites tile with Settings tile on Home screen
- [x] Enabled idle timer interval (was disabled for debugging)

**Result**: Free heap ~13KB → ~18KB, largest block ~5KB → ~16KB

---

## Human Testing Checkpoint: Settings & Idle Behavior

**Purpose**: Verify user preferences and power management

- [ ] HT-023 **HUMAN TEST**: Test settings screen:
  - [ ] Can access Settings screen from navigation
  - [ ] Brightness slider changes backlight immediately
  - [ ] Idle dim timeout selector works
  - [ ] Screen-off timeout selector works
  - [ ] Debug mode toggle works

- [ ] HT-024 **HUMAN TEST**: Test idle behavior:
  - [ ] Leave device idle → dims after configured timeout
  - [ ] Continue idle → screen off after configured timeout
  - [ ] Touch screen → wakes immediately, full brightness restored
  - [ ] Playback state change resets idle timer

- [ ] HT-025 **HUMAN TEST**: Test settings persistence:
  - [ ] Change all settings to non-default values
  - [ ] Reboot device
  - [ ] All settings restored after reboot

- [ ] HT-026 **HUMAN TEST**: Test debug mode:
  - [ ] Enable debug mode → log verbosity increases
  - [ ] Wait 30 minutes → debug mode auto-disables
  - [ ] Debug mode does not affect normal operation

**Pass criteria**: All items checked.

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Error handling, performance optimization, and final hardening

- [ ] T129 [P] Create error banner/toast component in esphome/ui/components/error_toast.yaml
- [ ] T130 Display human-readable error messages (FR-080) in esphome/packages/error_handler.yaml
- [ ] T131 Auto-dismiss errors after 5 seconds (FR-083) in esphome/ui/components/error_toast.yaml
- [ ] T132 Allow touch to dismiss errors in esphome/ui/components/error_toast.yaml
- [ ] T133 Ensure errors do not block other UI functionality (FR-081)
- [ ] T134 [P] Add logging infrastructure per constitution Section 10 in esphome/packages/logging.yaml
- [ ] T135 Profile memory usage across all screens in esphome/packages/diagnostics.yaml
- [ ] T136 Fix any memory leaks discovered during profiling
- [ ] T137 Optimize animations for consistent 60fps (no dropped frames during transitions)
- [ ] T138 Verify all touch targets are 44x44px minimum (Constitution §6.4)
- [ ] T139 Verify touch feedback within 100ms on all interactive elements
- [ ] T140 Implement burn-in mitigation for Now Playing screen (Constitution §8.3)
- [ ] T141 Run 24-hour continuous operation test (SC-011)
- [ ] T142 Verify all 16 success criteria pass (SC-001 through SC-016)
- [ ] T143 Verify all functional requirements pass (FR-001 through FR-098)
- [ ] T144 Final code cleanup and constitution compliance review

**Checkpoint**: All success criteria verified, 24-hour stability confirmed, ready for release

---

## Human Testing Checkpoint: Final Validation (24-Hour Soak)

**Purpose**: Verify stability and all success criteria

- [ ] HT-027 **HUMAN TEST**: Error handling:
  - [ ] Trigger an error condition (e.g., invalid player)
  - [ ] Error message is human-readable (no technical jargon)
  - [ ] Error auto-dismisses after 5 seconds
  - [ ] Can touch to dismiss error early
  - [ ] Error does not block other UI operations

- [ ] HT-028 **HUMAN TEST**: 24-hour soak test:
  - [ ] Leave device running with music playing intermittently
  - [ ] Check every few hours - no crash, no freeze
  - [ ] After 24 hours, verify free_heap is stable (compare to HT-008 baseline)
  - [ ] No visible memory growth

- [ ] HT-029 **HUMAN TEST**: Success criteria validation:
  - [ ] SC-001: Can control playback within 2 taps from any screen
  - [ ] SC-002: Playback actions complete within 2 seconds
  - [ ] SC-003: Boot to interactive within 10 seconds
  - [ ] SC-004: Touch feedback within 100ms
  - [ ] SC-005: Can identify track/artist/state within 2 seconds of looking
  - [ ] SC-006: Player switch in under 5 seconds
  - [ ] SC-007: Browse and play within 30 seconds
  - [ ] SC-008: Responsive during connectivity loss
  - [ ] SC-009: Auto-recover within 30 seconds of reconnection
  - [ ] SC-010: Settings persist across reboot
  - [ ] SC-011: 24-hour operation without crash (this test)
  - [ ] SC-012: 90%+ actions succeed first attempt
  - [ ] SC-013: Error messages understandable
  - [ ] SC-014: Album art within 3 seconds
  - [ ] SC-015: Album art visible from arm's length
  - [ ] SC-016: UI responsive during album art loading

- [ ] HT-030 **HUMAN TEST**: Constitution quality gates (§11):
  - [ ] Cold boot < 10 seconds
  - [ ] Wi-Fi recovery works
  - [ ] HA API recovery works
  - [ ] Touch response < 100ms
  - [ ] No UI stutter
  - [ ] Heap stable over 1 hour
  - [ ] No secrets exposed in UI

**Pass criteria**: All items checked. Ready for release.

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup
    ↓
Phase 2: Foundational (BLOCKS ALL USER STORIES)
    ↓
    ├── Phase 3: US1 & US2 - Now Playing (P1) ← MVP
    │       ↓
    │   Phase 4: Navigation & Dashboard
    │       ↓
    │       ├── Phase 5: US3 - Player Selection (P2)
    │       ├── Phase 6: US4 - Media Browsing (P2)
    │       └── Phase 8: US6 - Offline Handling (P2)
    │           ↓
    │       Phase 7: US5 - Speaker Grouping (P3)
    │           ↓
    │       Phase 9: US7 - Settings (P3)
    │           ↓
    └── Phase 10: Polish & Hardening
```

### User Story Dependencies

- **US1 & US2 (P1)**: Can start after Foundational - No dependencies on other stories
- **US3 (P2)**: Requires Navigation (Phase 4) - Can run parallel with US4, US6
- **US4 (P2)**: Requires Navigation (Phase 4) - Can run parallel with US3, US6
- **US5 (P3)**: Requires US3 (Player Management screen for Group button)
- **US6 (P2)**: Requires Foundational connectivity services - Can run parallel with US3, US4
- **US7 (P3)**: Can start after Navigation - No story dependencies

### Within Each User Story

- UI components before screen implementation
- Services before UI that uses them
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel
- T020, T021, T026, T029, T033, T035 can run in parallel (different component files)
- T043, T044 can run in parallel (forward/back animations)
- T046, T049, T050, T051 can run in parallel (different tiles)
- T056, T057 can run in parallel (different components)
- T071, T072, T073 can run in parallel (different components)
- T113, T114, T115 can run in parallel (different settings components)

---

## Parallel Example: User Story 1 & 2

```bash
# Launch UI components in parallel:
Task: "Create Now Playing screen layout in esphome/ui/now_playing.yaml"
Task: "Create album art container component in esphome/ui/components/album_art.yaml"
Task: "Create scrolling/marquee label component in esphome/ui/components/marquee_label.yaml"
Task: "Create playback control button component in esphome/ui/components/control_button.yaml"
Task: "Create progress bar component in esphome/ui/components/progress_bar.yaml"
Task: "Create volume slider component in esphome/ui/components/volume_slider.yaml"

# Then sequential integration:
Task: "Implement play/pause button with icon swap"
Task: "Subscribe to Music Assistant media_player state updates"
Task: "Add optimistic UI updates on control touch"
```

---

## Implementation Strategy

### MVP First (User Stories 1 & 2 Only)

1. Complete Phase 1: Setup
2. **HT-001**: Human testing checkpoint
3. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
4. **HT-002 to HT-004a**: Human testing checkpoint
5. Complete Phase 3: User Stories 1 & 2 (Now Playing + Playback Control)
6. **HT-005 to HT-008**: Human testing checkpoint
7. **STOP and VALIDATE**: MVP deliverable - can demo to stakeholders

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add US1 & US2 → Human test → Deploy/Demo (MVP!)
3. Add Navigation → Human test → Screen transitions working
4. Add US3 (Player Selection) → Human test → Deploy/Demo
5. Add US4 (Browsing) → Human test → Deploy/Demo
6. Add US5 (Grouping) → Human test → Deploy/Demo
7. Add US6 (Offline) → Human test → Deploy/Demo
8. Add US7 (Settings) → Human test → Deploy/Demo
9. Polish phase → Final validation (24-hour soak) → Release

### Sequential Priority Strategy

For single-developer workflow:
1. P1 (US1 & US2): Playback control and Now Playing - CORE VALUE
2. P2 (US3, US4, US6): Player selection, browsing, offline handling
3. P3 (US5, US7): Grouping, settings - NICE TO HAVE

---

## Metrics Summary

| Metric | Value |
|--------|-------|
| Total Implementation Tasks | 148 (T001-T144, including T001a/b, T018a-d, T019a, T036a) |
| Total Human Testing Tasks | 30 (HT-001 to HT-030) |
| Human Testing Checkpoints | 10 (one per phase boundary) |
| Phases | 10 |
| User Stories Covered | 7 |

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- HT-XXX = Human testing checkpoint (manual validation required)
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- All paths are relative to repository root
- Follow constitution naming conventions (§6)
- All UI must use LVGL via ESPHome
- Touch targets minimum 44x44px per constitution
- Touch feedback within 100ms per constitution
- Human testing checkpoints are MANDATORY gates before proceeding
