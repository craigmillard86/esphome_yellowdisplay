# Tasks: Album Songs Browse

**Input**: Design documents from `/specs/002-album-songs-browse/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, quickstart.md

**Tests**: Manual device testing only (ESPHome project - no automated tests)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **ESPHome project structure**:
  - `esphome/packages/` - Backend logic (browse.yaml, navigation.yaml)
  - `esphome/ui/` - UI definitions (browse_unified.yaml)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add new globals and extend category configuration

- [ ] T001 Add `browse_context_album_id` global (std::string) in esphome/packages/browse.yaml
- [ ] T002 Add `browse_context_album_name` global (std::string) in esphome/packages/browse.yaml
- [ ] T003 Add `browse_saved_category` global (int, initial -1) in esphome/packages/browse.yaml

---

## Phase 2: Foundational (Category 7 Infrastructure)

**Purpose**: Core category 7 handling that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T004 Add category 7 case to `browse_configure_page` script - set title to album name, hide alpha picker, show Play All button in esphome/packages/browse.yaml
- [ ] T005 Add category 7 case to `browse_category` script - initialize state and call API in esphome/packages/browse.yaml
- [ ] T006 Add category 7 HTTP request for tracks (media_type: track, album filter, order_by: track_number) in esphome/packages/browse.yaml
- [ ] T007 Add category 7 response parser - extract track_number and name, format as "N. Song Title" in esphome/packages/browse.yaml
- [ ] T008 Add category 7 to `browse_load_page` script for pagination support in esphome/packages/browse.yaml

**Checkpoint**: Foundation ready - category 7 can display songs (but no entry points yet)

---

## Phase 3: User Story 1+2 - View and Play Songs (Priority: P1) 🎯 MVP

**Goal**: Users can view songs in an album and tap to play individual songs

**Independent Test**: Navigate to Albums, tap album, verify songs display as "1. Song Name", tap song, verify playback starts

### Implementation for User Stories 1+2

- [x] T009 [US1] Create `browse_album_songs` script - saves album context, saves parent category/offset, configures page, loads tracks in esphome/packages/browse.yaml
- [x] T010 [US1] Update category 4 (Albums) item tap handler to call `browse_album_songs` instead of `play_media_item` in esphome/ui/browse_unified.yaml
- [x] T011 [US2] Verify category 7 item tap calls `play_media_item` then navigates to Now Playing (existing pattern) - verified: category 7 falls through to else case which calls play_media_item
- [x] T012 [US1] Add empty state handling for albums with no songs ("No songs in this album") in esphome/packages/browse.yaml
- [x] T013 [US2] Add no-player-selected error handling when tapping song without player in esphome/packages/browse.yaml - already implemented with logging (error toast disabled for memory)

**Checkpoint**: Albums → Album Songs → Play Song flow complete (MVP!)

---

## Phase 4: User Story 3 - Play All (Priority: P2)

**Goal**: Users can play entire album with single "Play All" tap

**Independent Test**: Navigate to album songs, tap Play All button, verify entire album queues and plays

### Implementation for User Story 3

- [x] T014 [US3] Create `play_album_all` script - calls music_assistant.play_media with album URI in esphome/packages/browse.yaml
- [x] T015 [US3] Update Play All button (btn_browse_play_all) on_click to call `play_album_all` when category is 7 in esphome/ui/browse_unified.yaml
- [x] T016 [US3] Verify Play All navigates to Now Playing after initiating playback in esphome/packages/browse.yaml

**Checkpoint**: Play All functionality complete

---

## Phase 5: User Story 4 - Artists Path (Priority: P2)

**Goal**: Users can access album songs from Artists → Artist Albums path

**Independent Test**: Navigate Artists → select artist → select album → verify album songs display

### Implementation for User Story 4

- [x] T017 [US4] Update category 6 (ArtistAlbums) item tap handler to call `browse_album_songs` instead of `play_media_item` in esphome/ui/browse_unified.yaml - completed in Phase 3, condition includes category 6
- [ ] T018 [US4] Test navigation depth: Artists → ArtistAlbums → AlbumSongs (3 levels, at nav stack limit) - requires manual device testing

**Checkpoint**: Artists → Artist Albums → Album Songs flow complete

---

## Phase 6: User Story 5 - Back Navigation (Priority: P2)

**Goal**: Back button correctly returns to parent category (Albums or Artist Albums)

**Independent Test**: Navigate to album songs from both paths, tap back, verify correct parent screen and scroll position

### Implementation for User Story 5

- [x] T019 [US5] Add category 7 case to `navigate_back` script - detect saved_category (4 or 6), restore offset in esphome/packages/navigation.yaml
- [x] T020 [US5] Clear album context (browse_context_album_id, browse_context_album_name) on back navigation in esphome/packages/navigation.yaml
- [x] T021 [US5] Reconfigure page for parent category and reload at saved offset in esphome/packages/navigation.yaml

**Checkpoint**: All navigation paths working correctly

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Edge cases, error handling, and final validation

- [x] T022 Handle long song titles with ellipsis truncation (existing long_mode: DOT should work) - uses existing browse item styling
- [x] T023 Handle pagination for albums with 5+ songs - uses existing browse_load_page pagination
- [x] T024 ESPHome compile test for original CYD (esphome compile esphome/main.yaml) - SUCCESS: RAM 14.8%, Flash 83.3%
- [x] T025 ESPHome compile test for Freenove S3 (esphome compile esphome/main_freenove_s3.yaml) - SUCCESS: RAM 16.3%, Flash 19.3%
- [ ] T026 Run quickstart.md validation checklist on physical device - requires manual testing
- [x] T027 Update navigation.yaml screen ID comments to include screen 13 if applicable - not needed (reuses page_browse)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - US1+2 (Phase 3) can proceed first (MVP)
  - US3 (Phase 4) depends on Phase 3 (needs song display working)
  - US4 (Phase 5) can proceed after Phase 3
  - US5 (Phase 6) depends on both Phase 4 and 5 (tests both paths)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **US1+2 (P1)**: Can start after Foundational (Phase 2) - Core MVP
- **US3 (P2)**: Can start after US1+2 - Needs song display working first
- **US4 (P2)**: Can start after US1+2 - Needs album songs screen working
- **US5 (P2)**: Depends on US3 and US4 - Tests back nav from both paths

### Within Each Phase

- Globals before scripts
- Scripts before UI handlers
- Core implementation before error handling
- Compile after each logical group

### Parallel Opportunities

- T001, T002, T003 can run in parallel (different globals)
- T004, T005 can run in parallel (different script functions)
- T024, T025 can run in parallel (different compile targets)

---

## Parallel Example: Phase 2 Foundation

```bash
# Can work on these scripts simultaneously:
Task T004: "Add category 7 case to browse_configure_page"
Task T005: "Add category 7 case to browse_category"

# Then sequentially:
Task T006: "Add HTTP request" (needs T005 context)
Task T007: "Add response parser" (needs T006)
Task T008: "Add browse_load_page" (needs T007)
```

---

## Implementation Strategy

### MVP First (User Stories 1+2 Only)

1. Complete Phase 1: Setup (3 tasks)
2. Complete Phase 2: Foundational (5 tasks)
3. Complete Phase 3: User Stories 1+2 (5 tasks)
4. **STOP and VALIDATE**: Test Albums → Album Songs → Play Song flow
5. Compile and test on device

### Incremental Delivery

1. Setup + Foundational → Category 7 infrastructure ready
2. Add US1+2 → Albums drill-down works → MVP!
3. Add US3 → Play All works
4. Add US4 → Artists path works
5. Add US5 → Back navigation works from all paths
6. Polish → Edge cases handled, compile verified

### Single Developer Strategy

Execute phases sequentially:
1. Phase 1 (15 min)
2. Phase 2 (45 min)
3. Phase 3 (30 min) → Test on device
4. Phase 4 (15 min)
5. Phase 5 (15 min)
6. Phase 6 (20 min)
7. Phase 7 (30 min) → Final validation

Estimated total: ~3 hours

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Compile after each phase to catch YAML/syntax errors early
- Test on physical device after Phase 3 (MVP checkpoint)
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
