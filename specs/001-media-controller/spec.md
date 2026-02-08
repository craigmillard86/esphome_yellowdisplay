# Feature Specification: Music Assistant Media Controller

**Feature Branch**: `001-media-controller`
**Created**: 2026-02-01
**Status**: Draft
**Input**: User description: "Dedicated, always-on physical media controller for Music Assistant on ESP32 CYD"

## Purpose & Scope

### Purpose

The system is a dedicated, always-on physical media controller for Music Assistant, providing fast, touch-based control of audio playback, content selection, and speaker grouping via Home Assistant.

It is designed to feel like a purpose-built appliance rather than a generic touchscreen dashboard.

### In Scope

- Controlling playback (play, pause, skip, seek, volume)
- Browsing and selecting audio content
- Selecting target players
- Creating and managing speaker groups
- Displaying now-playing information including album art
- Handling offline and degraded states gracefully

### Out of Scope

- Direct audio playback on the device
- Account management or Music Assistant configuration
- Advanced library management (tag editing, metadata fixes)
- Voice control

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Control Current Playback (Priority: P1)

As a household user, I want to control what's currently playing so I can pause, skip, or adjust volume without reaching for my phone or computer.

**Why this priority**: This is the core value proposition. Without playback control, the device serves no purpose. Users will interact with this functionality most frequently throughout the day.

**Independent Test**: Can be fully tested by playing music on any Music Assistant player, then using only the controller to pause, resume, skip tracks, and adjust volume. Delivers immediate, tactile media control.

**Acceptance Scenarios**:

1. **Given** music is playing on the active player, **When** I tap the pause button, **Then** playback pauses within 1 second and the button changes to show "play" state
2. **Given** music is paused, **When** I tap the play button, **Then** playback resumes within 1 second and the button changes to show "pause" state
3. **Given** music is playing, **When** I tap the next track button, **Then** the next track begins within 2 seconds and track information updates
4. **Given** music is playing, **When** I drag the volume slider, **Then** volume changes smoothly and the current level is displayed
5. **Given** music is playing, **When** I drag the progress bar, **Then** playback seeks to the selected position

---

### User Story 2 - View Now Playing Information (Priority: P1)

As a household user, I want to see what's currently playing at a glance—including album artwork—so I can identify the track visually without reading text.

**Why this priority**: Tied with playback control as core functionality. Album art provides instant visual recognition and makes the device feel like a premium music controller rather than a basic text display.

**Independent Test**: Can be fully tested by playing various tracks with and without album art, verifying metadata and artwork display correctly. Delivers immediate visual feedback about current audio.

**Acceptance Scenarios**:

1. **Given** music is playing, **When** I look at the screen, **Then** I can see the track title, artist name, and playback state
2. **Given** a track is playing with album information, **When** the track loads, **Then** the album name is displayed
3. **Given** a track has album art available, **When** the track loads, **Then** the album art is displayed prominently on the Now Playing screen
4. **Given** a track has no album art, **When** the track loads, **Then** a placeholder image is displayed instead
5. **Given** music is playing, **When** time passes, **Then** the progress bar updates smoothly showing elapsed and remaining time
6. **Given** the active player changes state, **When** the state updates, **Then** the display reflects the new state within 1 second
7. **Given** album art is loading, **When** the image is not yet available, **Then** a loading placeholder is shown briefly before the art appears

---

### User Story 3 - Select Target Player (Priority: P2)

As a household user with multiple speakers, I want to choose which player I'm controlling so I can manage audio in different rooms.

**Why this priority**: Essential for multi-room setups. Without player selection, the controller can only manage a single hardcoded player.

**Independent Test**: Can be fully tested by having multiple Music Assistant players available, switching between them, and verifying controls affect the selected player only.

**Acceptance Scenarios**:

1. **Given** multiple players are available, **When** I open the player selection screen, **Then** I see a list of all available players with their current status
2. **Given** I'm viewing the player list, **When** I tap a player, **Then** that player becomes the active target and I return to the Now Playing screen
3. **Given** a player is offline, **When** I view the player list, **Then** that player appears visually disabled and cannot be selected
4. **Given** I selected a player previously, **When** I restart the device, **Then** my last selected player is remembered and active

---

### User Story 4 - Browse and Play Content (Priority: P2)

As a household user, I want to browse my music library and start playing something new so I don't have to use another device to choose what to listen to.

**Why this priority**: Extends the controller from playback-only to a complete media selection device. Significantly increases utility.

**Independent Test**: Can be fully tested by navigating through browse categories, selecting content, and verifying playback starts on the active player.

**Acceptance Scenarios**:

1. **Given** I'm on the Now Playing screen, **When** I open the browse menu, **Then** I see entry points for browsing (recently played, favorites, playlists, albums, artists)
2. **Given** I'm browsing a category, **When** I scroll through the list, **Then** the interface remains responsive and loads content smoothly
3. **Given** I'm viewing a list of albums, **When** I tap an album, **Then** playback starts on the active player
4. **Given** I'm browsing, **When** I want to return to Now Playing, **Then** I can navigate back easily

---

### User Story 5 - Manage Speaker Groups (Priority: P3)

As a household user, I want to group and ungroup speakers so I can play synchronized audio across multiple rooms.

**Why this priority**: Advanced functionality for multi-room audio. Builds on player selection but requires more complex interaction.

**Independent Test**: Can be fully tested by creating a group from available players, verifying synchronized playback, then dissolving the group.

**Acceptance Scenarios**:

1. **Given** multiple players are available, **When** I open the grouping interface, **Then** I see all players with checkboxes to include/exclude them
2. **Given** I've selected multiple players, **When** I confirm the group, **Then** a synchronized group is created and becomes the active target
3. **Given** a player is in a group, **When** I remove it from the group, **Then** that player stops receiving synchronized audio
4. **Given** I'm creating a group, **When** I cancel, **Then** no changes are made and I return to the previous screen

---

### User Story 6 - Operate During Connectivity Issues (Priority: P2)

As a household user, I want the device to remain usable during network or service interruptions so I'm not left with a frozen or crashed screen.

**Why this priority**: Reliability is essential for an always-on appliance. Users must trust the device to handle real-world conditions.

**Independent Test**: Can be fully tested by disconnecting Wi-Fi or stopping Home Assistant, verifying the UI remains responsive and clearly indicates the issue.

**Acceptance Scenarios**:

1. **Given** the device is operating normally, **When** Wi-Fi connection is lost, **Then** the UI shows an "Offline" indicator and remains responsive
2. **Given** the device is offline, **When** controls are tapped, **Then** they are safely ignored or visually disabled (no crashes or freezes)
3. **Given** the device is offline, **When** connectivity is restored, **Then** the UI automatically recovers and shows current state within 30 seconds
4. **Given** Music Assistant becomes unavailable, **When** I view the screen, **Then** the last known state is displayed with a clear unavailable indicator

---

### User Story 7 - Adjust Device Settings (Priority: P3)

As a household user, I want to adjust screen brightness and idle behavior so the device fits my environment and doesn't disturb me at night.

**Why this priority**: Quality-of-life feature. Not essential for core operation but improves daily usability.

**Independent Test**: Can be fully tested by changing brightness settings, waiting for idle timeout, and verifying the screen dims/sleeps as configured.

**Acceptance Scenarios**:

1. **Given** I'm using the device, **When** I access settings, **Then** I can adjust screen brightness
2. **Given** I've set an idle timeout, **When** I don't touch the screen for the configured duration, **Then** the screen dims
3. **Given** the screen is dimmed or off, **When** I touch the screen, **Then** it wakes immediately and responds to my input
4. **Given** I've changed a setting, **When** I restart the device, **Then** my settings are preserved

---

### Edge Cases

- What happens when the active player is deleted or becomes permanently unavailable?
  - System prompts user to select a different player and clears the invalid selection
- What happens when track metadata is missing (no title, artist, or album)?
  - Display "Unknown" placeholders and continue functioning
- What happens when album art is unavailable or fails to load?
  - Display a generic placeholder image (e.g., music note icon) and continue functioning
- What happens when album art is very slow to load?
  - Show a loading placeholder, then display art when available; do not block other UI updates
- What happens when album art URL changes mid-track (e.g., metadata correction)?
  - Update to the new art smoothly without disrupting playback display
- What happens when the browse list is empty (no favorites, no recent)?
  - Show a helpful empty state message
- What happens when a group creation fails partially (some players join, others don't)?
  - Report which players failed and why, allow user to retry or accept partial group
- What happens when the user rapidly taps controls?
  - Debounce inputs to prevent service spam while still acknowledging the touch visually
- What happens during an over-the-air update?
  - Display update progress and prevent user interaction until complete

---

## Requirements *(mandatory)*

### Functional Requirements

#### System States

- **FR-001**: System MUST operate in one of five states: Booting, Connecting, Ready, Offline, or Error
- **FR-002**: System MUST display the current state visibly to the user at all times
- **FR-003**: System MUST transition between states based on connectivity and service availability
- **FR-004**: System MUST boot to a visible, interactive state within 10 seconds

#### Now Playing Display

- **FR-010**: System MUST display the current track title when music is playing
- **FR-011**: System MUST display the current artist name when music is playing
- **FR-012**: System MUST display the album name when available
- **FR-013**: System MUST display playback state (playing, paused, stopped)
- **FR-014**: System MUST display playback progress (elapsed and total time)
- **FR-015**: System MUST display the name of the active player or group
- **FR-016**: System MUST display connection status indicator
- **FR-017**: System MUST update playback state within 1 second of confirmed changes
- **FR-018**: System MUST update progress smoothly but rate-limited (2-4 updates per second)

#### Album Art Display

- **FR-090**: System MUST display album art on the Now Playing screen when available
- **FR-091**: System MUST display a placeholder image when album art is unavailable
- **FR-092**: System MUST display a loading indicator while album art is being fetched
- **FR-093**: System MUST NOT block UI updates while waiting for album art to load
- **FR-094**: Album art MUST be displayed at a size appropriate for the screen (readable from arm's length)
- **FR-095**: System MUST update album art when the track changes
- **FR-096**: System MUST cache recently displayed album art to reduce repeated fetches
- **FR-097**: System MUST handle album art fetch failures gracefully (fall back to placeholder)
- **FR-098**: Album art loading MUST NOT cause visible UI stutter or delay playback information updates

#### Playback Controls

- **FR-020**: Users MUST be able to play/pause the current track
- **FR-021**: Users MUST be able to skip to the next track
- **FR-022**: Users MUST be able to skip to the previous track
- **FR-023**: Users MUST be able to seek within the current track
- **FR-024**: Users MUST be able to adjust volume
- **FR-025**: Users MUST be able to mute/unmute audio
- **FR-026**: System MUST provide immediate visual feedback on control touch (within 100ms)
- **FR-027**: System MUST optimistically update UI on user action
- **FR-028**: System MUST reconcile optimistic updates with actual state within 2 seconds
- **FR-029**: System MUST debounce rapid control taps to prevent service spam

#### Player Selection

- **FR-030**: System MUST display a list of all available Music Assistant players
- **FR-031**: System MUST show player availability status (online/offline)
- **FR-032**: System MUST indicate the currently selected player
- **FR-033**: System MUST allow selection of any online player
- **FR-034**: System MUST prevent selection of offline players
- **FR-035**: System MUST persist the selected player across reboots
- **FR-036**: System MUST update all UI context when a player is selected

#### Speaker Grouping

- **FR-040**: System MUST display all players available for grouping
- **FR-041**: Users MUST be able to toggle players in/out of a group
- **FR-042**: Users MUST explicitly confirm group changes before they take effect
- **FR-043**: Users MUST be able to cancel group changes without applying them
- **FR-044**: System MUST report partial group creation failures clearly
- **FR-045**: System MUST prevent invalid grouping combinations
- **FR-046**: System MUST update the active target to the new group after creation

#### Media Browsing

- **FR-050**: System MUST provide browse entry points: recently played, favorites, playlists, albums, artists
- **FR-051**: System MUST support paginated or chunked loading for large lists
- **FR-052**: System MUST remain responsive during content loading
- **FR-053**: System MUST start playback on the active player when content is selected
- **FR-054**: System MUST communicate browsing/loading errors clearly
- **FR-055**: System MUST allow navigation back from any browse level

#### Offline & Degraded Operation

- **FR-060**: System MUST remain responsive when offline (UI never freezes)
- **FR-061**: System MUST display last known state when offline
- **FR-062**: System MUST disable or safely ignore controls when offline
- **FR-063**: System MUST NOT repeatedly retry failed service calls (no retry storms)
- **FR-064**: System MUST automatically attempt reconnection
- **FR-065**: System MUST update UI immediately when connectivity is restored

#### Settings

- **FR-070**: Users MUST be able to adjust screen brightness
- **FR-071**: Users MUST be able to configure idle dim timeout
- **FR-072**: Users MUST be able to configure screen off timeout
- **FR-073**: Users MUST be able to toggle debug mode
- **FR-074**: System MUST persist all settings across reboots
- **FR-075**: System MUST apply setting changes immediately

#### Error Handling

- **FR-080**: System MUST display human-readable error messages
- **FR-081**: System MUST NOT block unrelated functionality due to errors
- **FR-082**: System MUST surface persistent errors visibly
- **FR-083**: Errors MUST auto-dismiss after 5 seconds or on user touch

---

### Key Entities

- **Player**: A Music Assistant media player or speaker. Has name, availability status, current playback state, volume level, and group membership.

- **Group**: A collection of synchronized players. Has name, member players, and leader player.

- **Track**: The currently playing or queued audio item. Has title, artist, album, duration, and album art (image URL or reference).

- **Album Art**: Visual artwork associated with a track or album. Has image source (URL), loading state (loading/loaded/failed), and cached status.

- **Playback State**: The current state of audio output. Includes playing/paused/stopped status, current position, and volume.

- **System State**: The overall device operational state. One of: Booting, Connecting, Ready, Offline, Error.

- **User Settings**: Persisted user preferences. Includes brightness level, idle dim timeout, screen off timeout, debug mode flag.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can control playback (play/pause/skip) within 2 taps from any screen
- **SC-002**: Playback control actions complete within 2 seconds of user input
- **SC-003**: Device boots to interactive state within 10 seconds
- **SC-004**: Touch input receives visual feedback within 100 milliseconds
- **SC-005**: Users can identify current track, artist, and playback state at a glance (within 2 seconds of looking at screen)
- **SC-014**: Album art displays within 3 seconds of track change when art is available
- **SC-015**: Album art is visible and recognizable from typical viewing distance (arm's length, approximately 50cm)
- **SC-016**: UI remains responsive during album art loading (no visible stutter or freeze)
- **SC-006**: Users can switch between players in under 5 seconds
- **SC-007**: Users can browse and start playing content within 30 seconds
- **SC-008**: Device remains responsive during connectivity loss (no freezes or crashes)
- **SC-009**: Device automatically recovers from connectivity issues within 30 seconds of restoration
- **SC-010**: Settings changes persist across 100% of device reboots
- **SC-011**: Device operates continuously for 24+ hours without memory growth or crashes
- **SC-012**: 90% of user actions complete successfully on first attempt
- **SC-013**: Error messages are understandable by non-technical users

---

## Assumptions & Dependencies

### Assumptions

- Home Assistant is the single integration point for all media control
- Music Assistant exposes standard media_player entity behavior
- Network connectivity may be intermittent but is available during setup
- Users have already configured Music Assistant and Home Assistant
- The device will be mounted in a fixed location with consistent power
- Hardware variants exist within CYD family but share core display/touch capabilities
- Radio/streams availability depends on Music Assistant configuration (may not be present)

### Dependencies

- Home Assistant installation with native API enabled
- Music Assistant integration configured in Home Assistant
- At least one Music Assistant player available
- Wi-Fi network access
- Stable power supply

---

## Phase 2 Considerations (Future)

The following features are explicitly deferred to a future phase:

- **Queue Management**: View upcoming tracks, remove from queue, skip to specific track, clear queue
- **Advanced Browse Filters**: Search, genre filtering, shuffle modes
- **Browse List Thumbnails**: Album art in browse lists (Now Playing album art is Phase 1; browse thumbnails are deferred)
