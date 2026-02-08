# Implementation Plan: Music Assistant Media Controller

**Feature Branch**: `001-media-controller`
**Created**: 2026-02-01
**Status**: Draft
**Specification**: [spec.md](spec.md)
**Constitution**: [constitution.md](../../.specify/memory/constitution.md)

---

## Plan Overview

This plan translates the functional specification into implementable phases. Each phase is independently testable and delivers incremental value. The plan follows the constitution's principles of ESPHome-native design, LVGL UI, memory-conscious implementation, and graceful degradation.

**Total Phases**: 8
**Estimated Complexity**: High (multi-screen UI, real-time state sync, network resilience)

---

## Phase 1: Foundation & Hardware Bring-Up

### Objective

Establish a working ESPHome configuration that boots successfully on the ESP32-2432S028R hardware, initializes the display and touch controller, and renders a basic LVGL screen.

### Scope

- ESPHome base configuration with board definitions
- ILI9341 display initialization (320x240 landscape)
- XPT2046 touch controller initialization with calibration
- Basic LVGL setup with test screen
- Backlight control (PWM)
- Boot time measurement infrastructure

### Key Tasks

1. Create base ESPHome YAML with ESP32-2432S028R board configuration
2. Configure SPI bus for display and touch
3. Initialize ILI9341 display driver in landscape orientation
4. Initialize XPT2046 touch driver with calibration points
5. Configure PWM for backlight control
6. Create minimal LVGL boot screen with "Starting..." text
7. Add boot timing sensor to measure time-to-interactive
8. Validate touch input registers correctly

### Dependencies

- None (first phase)

### Validation Criteria

- [ ] Device boots without errors
- [ ] Display shows LVGL content in correct orientation (landscape)
- [ ] Touch input is detected and coordinates are accurate
- [ ] Backlight responds to brightness changes
- [ ] Boot to visible UI completes within 10 seconds (SC-003)
- [ ] No memory allocation failures during boot

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Touch calibration drift | Touch targets miss-aligned | Store calibration in NVS; provide recalibration option |
| Display initialization failure | No visual feedback | Add error LED indication; implement retry logic |
| SPI bus conflicts | Display or touch fails | Ensure proper CS pin management; test both peripherals |

### Constitution References

- Principle I: ESPHome-Native & LVGL Design
- Section 6: Display & Touch Interaction Rules (§6.1, §6.2, §6.3)
- Section 7: Backlight & Idle Management (§7.1)

### Specification References

- FR-004: Boot to visible, interactive state within 10 seconds
- SC-003: Device boots to interactive state within 10 seconds

---

## Phase 2: System States & Connectivity

### Objective

Implement the five-state system model (Booting, Connecting, Ready, Offline, Error) with visual indicators and automatic state transitions based on Wi-Fi and Home Assistant connectivity.

### Scope

- System state machine implementation
- Wi-Fi connection management
- Home Assistant Native API connection
- State indicator UI components
- Automatic reconnection logic
- Connection status persistence

### Key Tasks

1. Define system state enum and global state variable
2. Create state transition logic with event-driven updates
3. Implement Wi-Fi connection with timeout handling
4. Implement Home Assistant API connection with authentication
5. Create visual state indicator (icon + text) for header area
6. Add offline detection with debounce (avoid flapping)
7. Implement automatic reconnection with exponential backoff
8. Create "Connecting" screen with progress indication
9. Ensure UI remains responsive during connection attempts

### Dependencies

- Phase 1: Hardware initialization complete

### Validation Criteria

- [ ] System transitions through Booting → Connecting → Ready on normal boot
- [ ] Offline state is detected within 5 seconds of Wi-Fi loss
- [ ] UI remains responsive during Offline state (FR-060)
- [ ] Automatic reconnection succeeds within 30 seconds of restoration (SC-009)
- [ ] State indicator is visible on all screens (FR-002)
- [ ] No retry storms during prolonged outages (FR-063)

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Wi-Fi instability | Frequent state changes | Debounce connection status; avoid UI flicker |
| HA API timeout | Stuck in Connecting | Implement connection timeout; transition to Offline |
| Memory leak in reconnection | Crash after prolonged offline | Use static buffers; test 24-hour offline scenario |

### Constitution References

- Principle V: Graceful Degradation
- Section 2: Architecture Boundaries (§2.1, §2.2)
- Section 9: Music Assistant UX Rules (§9.5)

### Specification References

- FR-001, FR-002, FR-003: System state requirements
- FR-060 through FR-065: Offline & degraded operation
- SC-008: Device remains responsive during connectivity loss
- SC-009: Automatic recovery within 30 seconds

---

## Phase 3: Now Playing UI & Core Playback Control

### Objective

Implement the Now Playing screen with track metadata display, album art, playback controls, progress bar, and volume slider. This is the primary user interface and core value proposition.

### Scope

- Now Playing screen layout (split horizontal per UI spec)
- Track metadata display (title, artist, album)
- Album art display with placeholder and loading states
- Playback controls (play/pause, previous, next)
- Progress bar with seek capability
- Volume slider
- Optimistic UI updates with reconciliation
- Touch feedback within 100ms

### Key Tasks

1. Create Now Playing screen with split layout (40% left / 60% right)
2. Implement album art container with rounded corners and shadow
3. Add placeholder image for missing album art
4. Add loading indicator for album art fetching
5. Implement album art caching strategy
6. Create scrolling/marquee labels for long track titles
7. Implement playback control buttons (44x44px minimum)
8. Add play/pause state toggle with icon swap
9. Create progress bar with touch-to-seek
10. Create volume slider with smooth dragging
11. Implement debounced API calls for slider releases (FR-029)
12. Add optimistic UI updates on control touch (FR-027)
13. Implement state reconciliation within 2 seconds (FR-028)
14. Subscribe to Music Assistant media_player state updates

### Dependencies

- Phase 2: Home Assistant connectivity established

### Validation Criteria

- [ ] Track title, artist, album display correctly (FR-010, FR-011, FR-012)
- [ ] Album art displays within 3 seconds of track change (SC-014)
- [ ] Placeholder shows when art unavailable (FR-091)
- [ ] Loading indicator shows during art fetch (FR-092)
- [ ] UI does not stutter during art loading (FR-098, SC-016)
- [ ] Play/pause responds within 1 second (User Story 1, Scenario 1)
- [ ] Skip track completes within 2 seconds (User Story 1, Scenario 3)
- [ ] Volume changes smoothly without jumping (FR-024)
- [ ] Progress bar updates 2-4 times per second (FR-018)
- [ ] Touch feedback within 100ms (SC-004)
- [ ] All touch targets are 44x44px minimum (Constitution §6.4)

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Album art fetch timeout | Blank or stuck loading | 3-second timeout; fall back to placeholder |
| Large album art crashes ESP32 | Memory exhaustion | Limit image size; use LVGL image decoder with size cap |
| State sync lag | UI shows stale data | Optimistic updates; periodic poll as backup |
| Rapid button taps | API spam | Debounce with 200ms minimum interval |

### Constitution References

- Principle III: Responsive UI
- Principle VIII: Design-First UI Implementation
- Section 6: Display & Touch Interaction Rules (§6.4, §6.5, §6.6)
- Section 9: Music Assistant UX Rules (§9.1, §9.2, §9.3, §9.4)

### Specification References

- FR-010 through FR-018: Now Playing display requirements
- FR-020 through FR-029: Playback control requirements
- FR-090 through FR-098: Album art requirements
- SC-001, SC-002, SC-004, SC-005: Playback UX criteria
- SC-014, SC-015, SC-016: Album art criteria
- User Story 1: Control Current Playback
- User Story 2: View Now Playing Information

---

## Phase 4: Home Dashboard & Navigation

### Objective

Implement the Home Dashboard as the central navigation hub with a 2x2 tile grid, and establish the screen navigation system with animated transitions.

### Scope

- Home Dashboard screen with 2x2 tile layout
- Navigation tiles: Now Playing, Library, Speakers, Favorites
- Screen manager for navigation between screens
- Animated screen transitions (slide left/right)
- Back button pattern for sub-screens
- Header bar with status indicators

### Key Tasks

1. Create Home Dashboard screen with 2x2 grid layout
2. Design tile components with icons and status text
3. Implement Now Playing tile with mini-thumbnail and current song
4. Implement Library tile with icon
5. Implement Speakers tile with dynamic active count
6. Implement Favorites tile with icon
7. Add press state styling (border highlight) for tiles
8. Implement screen manager with navigation stack
9. Add slide-left animation for forward navigation
10. Add slide-right animation for back navigation
11. Create consistent header bar component
12. Add back button to all sub-screens
13. Implement tap-anywhere on Lock Screen to navigate to Home

### Dependencies

- Phase 3: Now Playing screen available as navigation target

### Validation Criteria

- [ ] Home Dashboard displays 2x2 tile grid
- [ ] Tiles have 10px rounded corners per UI spec
- [ ] Tile press provides visual feedback within 100ms
- [ ] Navigation to Now Playing works from tile tap
- [ ] Screen transitions complete in 200-300ms
- [ ] Back navigation returns to previous screen
- [ ] Speakers tile shows accurate active count
- [ ] Navigation does not leak memory (test 100 transitions)

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Animation stutters | Poor perceived performance | Use LVGL built-in animations; avoid custom frame logic |
| Screen stack overflow | Crash on deep navigation | Limit stack depth; use flat navigation model |
| Tile status stale | Misleading information | Subscribe to relevant entity updates |

### Constitution References

- Principle III: Responsive UI (60fps target)
- Principle VIII: Design-First UI Implementation
- Section 6: Display & Touch Interaction Rules (§6.5)

### Specification References

- UI Technical Spec: Screen B (Home Dashboard)
- UI Technical Spec: Section 4.1 (Navigation Logic)

---

## Phase 5: Player Selection

### Objective

Implement the Player Management screen allowing users to view all available Music Assistant players, see their status, and select which player to control.

### Scope

- Player Management screen with scrollable list
- Player row display (icon, name, status, volume)
- Online/offline visual distinction
- Player selection interaction
- Active player persistence across reboots
- Volume slider per player row

### Key Tasks

1. Create Player Management screen layout
2. Implement scrollable list container for players
3. Create player row component with icon, label, status
4. Add volume slider to each player row (50% width)
5. Add power toggle switch per player
6. Query Music Assistant for available players
7. Subscribe to player state updates
8. Implement online/offline styling (active vs greyed)
9. Add tap-to-select behavior for player rows
10. Prevent selection of offline players (FR-034)
11. Update header to show selected player name
12. Persist selected player ID to NVS (FR-035)
13. Restore selected player on boot
14. Handle deleted/unavailable player gracefully

### Dependencies

- Phase 4: Navigation system in place
- Phase 2: Home Assistant API connectivity

### Validation Criteria

- [ ] All available players appear in list (FR-030)
- [ ] Player status (online/offline) is accurate (FR-031)
- [ ] Selected player is visually indicated (FR-032)
- [ ] Offline players cannot be selected (FR-034)
- [ ] Selection persists across reboot (FR-035)
- [ ] Player switch completes in under 5 seconds (SC-006)
- [ ] Volume changes affect selected player only
- [ ] Invalid saved player prompts re-selection

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Many players slow list | UI lag | Limit displayed players; implement lazy loading |
| Player disappears mid-session | Controls fail | Detect invalid player; prompt re-selection |
| NVS write failure | Lost preference | Verify write; retry on failure |

### Constitution References

- Principle V: Graceful Degradation
- Section 9: Music Assistant UX Rules (§9.5)

### Specification References

- FR-030 through FR-036: Player selection requirements
- SC-006: Switch between players in under 5 seconds
- User Story 3: Select Target Player

---

## Phase 6: Media Browsing & Selection

### Objective

Implement the Library browsing interface allowing users to navigate content categories, scroll through lists, and start playback of selected items.

### Scope

- Music Library Hub screen with category entry points
- Browse categories: recently played, favorites, playlists, albums, artists
- Scrollable content lists
- Content selection to start playback
- Back navigation through browse hierarchy
- Loading states and empty states
- Search interface with keyboard

### Key Tasks

1. Create Music Library Hub screen with category tiles
2. Implement Recently Played list view
3. Implement Favorites list view (synced from MA)
4. Implement Playlists list view
5. Implement Albums list view
6. Implement Artists list view
7. Create list item component (text only, per Phase 1 scope)
8. Implement paginated/chunked loading for large lists (FR-051)
9. Add loading indicator during content fetch
10. Add empty state message for empty categories
11. Implement tap-to-play on list items (FR-053)
12. Create Search screen with text input area
13. Implement QWERTY keyboard (30x30px keys per UI spec)
14. Link keyboard to textarea for search input
15. Implement search query submission
16. Display search results in overlay/dropdown
17. Add navigation breadcrumbs or back button

### Dependencies

- Phase 4: Navigation system in place
- Phase 3: Playback initiation mechanism

### Validation Criteria

- [ ] All five browse categories accessible (FR-050)
- [ ] Lists scroll smoothly without stutter (FR-052)
- [ ] Large lists load in chunks without blocking UI (FR-051)
- [ ] Selecting content starts playback on active player (FR-053)
- [ ] Empty categories show helpful message
- [ ] Loading errors display clearly (FR-054)
- [ ] Back navigation works from any browse depth (FR-055)
- [ ] Search keyboard is usable (keys at least 30x30px)
- [ ] Browse and play completes within 30 seconds (SC-007)

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Large library overwhelms memory | Crash | Strict pagination; limit items in memory |
| Slow API response | Perceived hang | Show loading spinner; implement timeout |
| Keyboard blocks screen | Poor UX | Position keyboard at bottom; results above |

### Constitution References

- Principle IV: Memory-Conscious Implementation
- Section 6: Display & Touch Interaction Rules

### Specification References

- FR-050 through FR-055: Media browsing requirements
- SC-007: Browse and start playing within 30 seconds
- User Story 4: Browse and Play Content
- UI Technical Spec: Screen F (Search & Keyboard)

---

## Phase 7: Speaker Grouping

### Objective

Implement the multi-room grouping interface allowing users to create, modify, and dissolve speaker groups for synchronized playback.

### Scope

- Multi-room Grouping screen with checklist
- Player checkboxes for group membership
- Group creation/modification confirmation
- Partial failure handling and reporting
- Cancel without applying changes
- Active target update after group creation

### Key Tasks

1. Create Multi-room Grouping screen with checklist layout
2. Display all players with checkboxes
3. Pre-check players already in current group
4. Store checkbox states locally before apply (bitmask/array)
5. Implement Apply button at bottom
6. On Apply: validate selections, send group command
7. Handle partial group creation failure (FR-044)
8. Display which players failed and why
9. Implement Cancel button to discard changes (FR-043)
10. Update active target to new group after creation (FR-046)
11. Prevent invalid grouping combinations (FR-045)
12. Add Group button to Player Management header

### Dependencies

- Phase 5: Player list and selection in place

### Validation Criteria

- [ ] All players appear with checkboxes (FR-040)
- [ ] Checkboxes toggle correctly (FR-041)
- [ ] Apply requires explicit confirmation (FR-042)
- [ ] Cancel discards changes (FR-043)
- [ ] Partial failures are reported clearly (FR-044)
- [ ] New group becomes active target (FR-046)
- [ ] Invalid combinations are prevented (FR-045)

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Group command fails silently | User confusion | Verify response; show success/failure |
| Partial group is worse than none | Inconsistent playback | Offer retry or rollback option |
| Checkbox state out of sync | Wrong players grouped | Refresh state before showing screen |

### Constitution References

- Principle V: Graceful Degradation
- Section 9: Music Assistant UX Rules

### Specification References

- FR-040 through FR-046: Speaker grouping requirements
- User Story 5: Manage Speaker Groups
- UI Technical Spec: Screen E (Multi-Room Grouping)

---

## Phase 8: Settings, Polish & Performance Hardening

### Objective

Implement device settings, error handling patterns, idle behavior, and perform comprehensive testing and optimization to meet all quality and reliability requirements.

### Scope

- Settings screen with brightness, timeouts, debug toggle
- Settings persistence to NVS
- Idle dim and screen-off behavior
- Wake-on-touch from dimmed/off state
- Error display system (toast/banner)
- Error auto-dismiss behavior
- Memory profiling and leak detection
- 24-hour stability testing
- Performance optimization

### Key Tasks

1. Create Device Settings screen layout
2. Implement brightness slider with immediate apply (FR-070, FR-075)
3. Implement idle dim timeout selector (FR-071)
4. Implement screen-off timeout selector (FR-072)
5. Implement debug mode toggle (FR-073)
6. Persist all settings to NVS (FR-074)
7. Load settings on boot
8. Implement idle timer based on touch activity
9. Dim backlight after idle timeout
10. Turn off backlight after screen-off timeout
11. Wake immediately on touch from dimmed/off (User Story 7, Scenario 3)
12. Create error banner/toast component
13. Display human-readable error messages (FR-080)
14. Auto-dismiss errors after 5 seconds (FR-083)
15. Allow touch to dismiss errors (FR-083)
16. Ensure errors don't block other functionality (FR-081)
17. Run 24-hour continuous operation test (SC-011)
18. Profile memory usage; fix any leaks
19. Optimize animations for consistent 60fps
20. Verify all success criteria pass

### Dependencies

- All previous phases complete

### Validation Criteria

- [ ] Brightness setting persists across reboot (FR-074)
- [ ] Idle dim activates after configured timeout
- [ ] Screen off activates after configured timeout
- [ ] Touch wakes screen immediately (User Story 7, Scenario 3)
- [ ] Settings persist across 100% of reboots (SC-010)
- [ ] Error messages are understandable (SC-013)
- [ ] Errors auto-dismiss after 5 seconds (FR-083)
- [ ] Device runs 24+ hours without crash or memory growth (SC-011)
- [ ] 90% of user actions succeed on first attempt (SC-012)
- [ ] All functional requirements pass
- [ ] All success criteria pass

### Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Memory leak discovered late | Major rework | Profile early; test overnight in each phase |
| NVS wear | Settings corruption | Minimize write frequency; use wear-leveling |
| Idle timer conflicts with playback | Dims during use | Reset timer on state changes, not just touch |

### Constitution References

- Principle IV: Memory-Conscious Implementation
- Principle VI: Testable Configuration
- Section 7: Backlight & Idle Management (§7.1, §7.2, §7.3, §7.4)
- Section 11: Quality Gates

### Specification References

- FR-070 through FR-075: Settings requirements
- FR-080 through FR-083: Error handling requirements
- SC-010: Settings persist across reboots
- SC-011: 24-hour continuous operation
- SC-012: 90% first-attempt success
- SC-013: Understandable error messages
- User Story 7: Adjust Device Settings

---

## Milestone Checklist

### Phase 1: Foundation & Hardware Bring-Up
- [ ] ESPHome compiles and uploads successfully
- [ ] Display shows content in landscape orientation
- [ ] Touch input is detected accurately
- [ ] Backlight responds to brightness control
- [ ] Boot time under 10 seconds

### Phase 2: System States & Connectivity
- [ ] Five system states implemented and transitionable
- [ ] Wi-Fi connection established automatically
- [ ] Home Assistant API connected with auth
- [ ] State indicator visible on screen
- [ ] Offline detection and recovery working

### Phase 3: Now Playing UI & Core Playback Control
- [ ] Track metadata displays correctly
- [ ] Album art displays within 3 seconds
- [ ] Placeholder shows for missing art
- [ ] Play/pause/skip controls functional
- [ ] Progress bar updates and is seekable
- [ ] Volume slider works smoothly
- [ ] Touch feedback within 100ms

### Phase 4: Home Dashboard & Navigation
- [ ] 2x2 tile grid displays correctly
- [ ] Navigation to all screens works
- [ ] Screen transitions animate smoothly
- [ ] Back navigation works consistently

### Phase 5: Player Selection
- [ ] Player list displays all available players
- [ ] Online/offline status accurate
- [ ] Player selection changes active target
- [ ] Selection persists across reboot

### Phase 6: Media Browsing & Selection
- [ ] All browse categories accessible
- [ ] Lists scroll smoothly
- [ ] Content selection starts playback
- [ ] Search keyboard functional
- [ ] Empty states display helpfully

### Phase 7: Speaker Grouping
- [ ] Grouping interface displays all players
- [ ] Checkboxes toggle correctly
- [ ] Group creation works
- [ ] Partial failures reported clearly
- [ ] Cancel discards changes

### Phase 8: Settings, Polish & Performance Hardening
- [ ] All settings adjustable and persistent
- [ ] Idle dim and screen-off functional
- [ ] Wake-on-touch works
- [ ] Errors display and auto-dismiss
- [ ] 24-hour stability test passed
- [ ] All success criteria verified

---

## Cross-Cutting Concerns

The following concerns apply across all phases:

### Memory Management
- Monitor heap usage in each phase
- Set memory budgets per component
- Test with constrained memory conditions
- No dynamic allocations in render loop

### Touch Responsiveness
- All interactive elements 44x44px minimum
- Visual feedback within 100ms
- Debounce rapid inputs

### Offline Behavior
- Test each phase with connectivity loss
- Verify graceful degradation
- No crashes or freezes

### Code Quality
- Follow constitution naming conventions (Section 5)
- Use modular YAML packages (Constitution §2.3)
- Document public interfaces
- Maintain separation of concerns

---

## Phase 2 Features (Deferred)

The following features are explicitly out of scope for this plan and deferred to a future phase:

- **Queue Management**: View upcoming tracks, remove from queue, skip to specific track, clear queue
- **Advanced Browse Filters**: Genre filtering, shuffle modes
- **Browse List Thumbnails**: Album art in browse lists (Now Playing album art is in scope)
- **Lock Screen**: Clock and weather display (lower priority than core functionality)

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02-01 | Initial plan created |
