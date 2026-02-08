# Feature Specification: Album Songs Browse

**Feature Branch**: `002-album-songs-browse`
**Created**: 2026-02-08
**Status**: Draft
**Input**: User description: "Enhance browse section to enable clicking albums and showing songs from that album to pick from, should also be able to play all songs in the album. The same screen should be available in both artists and albums"

## Clarifications

### Session 2026-02-08

- Q: What information should be displayed for each song in the list? → A: Track number + title only (e.g., "1. Song Name")

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Songs in an Album (Priority: P1)

As a user browsing my music library, I want to tap on an album to see all songs within that album, so I can select a specific song to play.

**Why this priority**: This is the core functionality - users need to see album contents before they can select individual songs. Without this, the feature has no value.

**Independent Test**: Can be fully tested by navigating to any album (from Artists view or Albums browse), tapping it, and verifying the song list displays correctly with song titles visible.

**Acceptance Scenarios**:

1. **Given** I am viewing an album list (either from Artists view or Albums browse), **When** I tap on an album, **Then** I see a screen showing all songs in that album with their titles displayed
2. **Given** I am on the album songs screen, **When** the songs load, **Then** I see song titles in a scrollable list ordered by track number
3. **Given** I am on the album songs screen, **When** I look at the header, **Then** I see the album name and a back button to return to the previous screen

---

### User Story 2 - Play Individual Song from Album (Priority: P1)

As a user viewing songs in an album, I want to tap on a song to start playing it, so I can listen to a specific track from the album.

**Why this priority**: Equal priority with viewing songs - playing a song is the primary action users will take after seeing the song list.

**Independent Test**: Can be fully tested by tapping any song in the album songs list and verifying playback starts on the selected player.

**Acceptance Scenarios**:

1. **Given** I am viewing the songs in an album, **When** I tap on a song, **Then** that song starts playing on the currently selected media player
2. **Given** I tap on a song, **When** playback starts, **Then** I am navigated to the Now Playing screen showing the song details
3. **Given** no media player is selected, **When** I tap on a song, **Then** I see an error message indicating I need to select a player first

---

### User Story 3 - Play All Songs in Album (Priority: P2)

As a user viewing an album, I want to play all songs in the album with a single action, so I can enjoy the entire album without selecting each song individually.

**Why this priority**: Important for album listening experience but secondary to individual song selection which is more frequently used.

**Independent Test**: Can be fully tested by tapping the "Play All" button and verifying all album songs are queued and playback begins.

**Acceptance Scenarios**:

1. **Given** I am viewing the songs in an album, **When** I tap the "Play All" button, **Then** all songs from the album are queued and playback begins with the first track
2. **Given** I tap "Play All", **When** playback starts, **Then** I am navigated to the Now Playing screen
3. **Given** the album has multiple songs, **When** I tap "Play All", **Then** the songs play in track order

---

### User Story 4 - Access Album Songs from Artists Browse (Priority: P2)

As a user browsing by artist, I want to tap on an album within an artist's discography to see that album's songs, so I can navigate my library by artist and then drill down to specific albums.

**Why this priority**: Provides the same album songs functionality from the artist navigation path, ensuring consistent experience across browse methods.

**Independent Test**: Can be fully tested by navigating Artists > Artist > Album and verifying the album songs screen appears with correct songs.

**Acceptance Scenarios**:

1. **Given** I am viewing an artist's albums, **When** I tap on an album, **Then** I see the album songs screen for that album
2. **Given** I navigated through Artists > Artist > Album Songs, **When** I tap the back button, **Then** I return to the artist's album list

---

### User Story 5 - Access Album Songs from Albums Browse (Priority: P2)

As a user browsing albums directly, I want to tap on any album to see its songs, so I can quickly access album contents without navigating through artists.

**Why this priority**: Alternative navigation path - same functionality as Story 4 but from the Albums browse section.

**Independent Test**: Can be fully tested by navigating Albums > Album and verifying the album songs screen appears.

**Acceptance Scenarios**:

1. **Given** I am in the Albums browse section, **When** I tap on an album, **Then** I see the album songs screen for that album
2. **Given** I navigated through Albums > Album Songs, **When** I tap the back button, **Then** I return to the Albums browse list

---

### Edge Cases

- What happens when an album has no songs? Display an empty state message "No songs in this album"
- What happens when loading songs fails? Display an error toast and allow retry
- What happens when the album has many songs (20+)? Songs should be scrollable in the list
- What happens if the song title is very long? Truncate with ellipsis to fit the display width
- What happens if playback fails? Show error toast with the failure reason

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST display a list of songs when a user taps on an album from either the Artists or Albums browse sections
- **FR-002**: System MUST show the album name in the header of the album songs screen
- **FR-003**: System MUST provide a back button to return to the previous browse screen
- **FR-004**: System MUST display songs with track number and title (format: "1. Song Name") in track order
- **FR-005**: System MUST allow scrolling when the song list exceeds the visible screen area
- **FR-006**: System MUST start playback of the selected song when a user taps on a song
- **FR-007**: System MUST navigate to the Now Playing screen after starting playback
- **FR-008**: System MUST provide a "Play All" action that queues all album songs and starts playback
- **FR-009**: System MUST display an error message if playback is attempted without a selected media player
- **FR-010**: System MUST display an appropriate message when an album contains no songs
- **FR-011**: System MUST handle long song titles by truncating with ellipsis
- **FR-012**: System MUST use the same album songs screen design regardless of navigation path (Artists or Albums)

### Key Entities

- **Album**: Represents a music album with name, artist, and a collection of songs
- **Song/Track**: Represents an individual track within an album with title, track number, and playback identifier
- **Media Player**: The target device for audio playback (already exists in system)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can navigate from album selection to viewing songs in under 2 seconds
- **SC-002**: Users can start playing a song with a single tap after viewing the song list
- **SC-003**: "Play All" functionality queues the entire album and starts playback within 3 seconds
- **SC-004**: Album songs screen displays correctly for albums with 1-50 songs
- **SC-005**: Navigation back to previous screen works correctly 100% of the time
- **SC-006**: Same album songs screen is accessible from both Artists and Albums browse paths

## Assumptions

- The existing browse infrastructure (Artists, Albums sections) is functional and will be extended
- Music Assistant provides an API/method to retrieve songs for a given album
- The media player selection functionality already exists and works correctly
- Track ordering information (track number) is available from the music source
- The current navigation stack implementation supports adding another level of depth
