# Tasks: Artist Album Browse

**Input**: Design documents from `/specs/001-artist-album-browse/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Manual functional testing only (per Constitution §11.2) - no automated tests for embedded firmware.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

ESPHome package structure:
- **Packages**: `esphome/packages/` - Backend logic and scripts
- **UI**: `esphome/ui/` - LVGL screen definitions
- **Main**: `esphome/main.yaml` - Entry point (no changes needed)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Add global variables and category constant needed by all user stories

- [x] T001 Add `browse_context_artist_id` global (std::string) in `esphome/packages/browse.yaml`
- [x] T002 Add `browse_context_artist_name` global (std::string) in `esphome/packages/browse.yaml`
- [x] T003 Add `browse_saved_offset` global (int) in `esphome/packages/browse.yaml`
- [x] T004 Document category constant 6 = ArtistAlbums in browse.yaml header comment

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T005 Add category 6 case in `browse_load_page` script with artist filter parameter in `esphome/packages/browse.yaml`
- [x] T006 Add `browse_artist_albums` script (parameters: artist_id, artist_name) in `esphome/packages/browse.yaml`

**Checkpoint**: Foundation ready - browse can now fetch albums filtered by artist

---

## Phase 3: User Story 1 - Browse Albums by Artist (Priority: P1) 🎯 MVP

**Goal**: Tap an artist in Artists list to see their albums with pagination and back navigation

**Independent Test**: Navigate to Artists → tap artist → verify albums display with artist name in header → tap back → verify return to Artists at same position

### Implementation for User Story 1

- [x] T007 [US1] Modify artist item tap handlers to call `browse_artist_albums` instead of `play_media_item` when `current_browse_category == 5` in `esphome/ui/browse_unified.yaml`
- [x] T008 [US1] Update header title display logic to show artist name when `current_browse_category == 6` in `esphome/ui/browse_unified.yaml`
- [x] T009 [US1] Hide alpha picker when `current_browse_category == 6` in `esphome/ui/browse_unified.yaml`
- [x] T010 [US1] Modify `navigate_back` script to handle category 6 → restore saved offset and reload Artists in `esphome/packages/navigation.yaml`
- [x] T011 [US1] Add "No albums" message display when album list is empty for category 6 in `esphome/ui/browse_unified.yaml`
- [x] T012 [US1] Verify pagination works for artist albums (existing prev/next should work automatically)

**Checkpoint**: User Story 1 complete - can browse artist albums and navigate back

---

## Phase 4: User Story 2 - Play All Music by Artist (Priority: P2)

**Goal**: Add "Play All" button to play all tracks by selected artist

**Independent Test**: From artist albums screen → tap Play All → verify playback starts → verify navigation to Now Playing

### Implementation for User Story 2

- [x] T013 [P] [US2] Add `play_artist_all` script using `music_assistant.play_media` with artist URI in `esphome/packages/browse.yaml`
- [x] T014 [US2] Add Play All button widget (hidden by default) to browse header in `esphome/ui/browse_unified.yaml`
- [x] T015 [US2] Show/hide Play All button based on `current_browse_category == 6` in `esphome/ui/browse_unified.yaml`
- [x] T016 [US2] Wire Play All button `on_click` to call `play_artist_all` script in `esphome/ui/browse_unified.yaml`
- [x] T017 [US2] Navigate to Now Playing screen after playback initiated in `play_artist_all` script

**Checkpoint**: User Story 2 complete - can play all music by artist from header button

---

## Phase 5: User Story 3 - Play Individual Album from Artist View (Priority: P3)

**Goal**: Tap an album in artist albums view to start playing it

**Independent Test**: From artist albums screen → tap album → verify album playback starts → verify navigation to Now Playing

### Implementation for User Story 3

- [x] T018 [US3] Verify album tap handlers call `play_media_item` when `current_browse_category == 6` in `esphome/ui/browse_unified.yaml`
- [x] T019 [US3] Verify navigation to Now Playing after album playback (should use existing behavior)
- [x] T020 [US3] Test album playback with various artists and album counts

**Checkpoint**: All user stories complete - full artist → album → play flow working

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Edge cases, error handling, and verification

- [ ] T021 Test artist with 0 albums - verify "No albums" message and Play All still works
- [ ] T022 Test artist with 50+ albums - verify pagination works correctly
- [ ] T023 Test rapid back button presses - verify no crashes or navigation issues
- [ ] T024 Test with WiFi disconnected - verify offline state handled gracefully
- [ ] T025 Monitor heap usage during repeated artist → albums → back navigation for 15 minutes
- [ ] T026 Verify cold boot still completes in <10 seconds (Constitution §11.2)
- [x] T027 Verify touch feedback <100ms on all new buttons (Constitution §1.2)
- [x] T028 Update browse.yaml file header comment with new functionality description

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-5)**: All depend on Foundational phase completion
  - User stories should be done in priority order (P1 → P2 → P3)
  - P1 (US1) must complete before P2 (US2) because Play All requires artist context
  - P3 (US3) is mostly verification of existing behavior
- **Polish (Phase 6)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Depends on US1 (needs artist context from drill-down navigation)
- **User Story 3 (P3)**: Can technically start after Foundational, but builds on US1 navigation

### Within Each User Story

- UI changes should follow script implementations
- Test at each checkpoint before proceeding

### Parallel Opportunities

- T001, T002, T003 can run in parallel (different globals in same file section)
- T013 can run in parallel with T014-T016 (different files)
- T021-T024 can run in parallel (independent test scenarios)

---

## Parallel Example: Setup Phase

```bash
# Add all globals together (same file section, no conflicts):
T001: Add browse_context_artist_id global
T002: Add browse_context_artist_name global
T003: Add browse_saved_offset global
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T004)
2. Complete Phase 2: Foundational (T005-T006)
3. Complete Phase 3: User Story 1 (T007-T012)
4. **STOP and VALIDATE**: Test artist drill-down independently
5. Flash and test on device

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Flash (MVP - can browse artist albums!)
3. Add User Story 2 → Test independently → Flash (adds Play All)
4. Add User Story 3 → Test independently → Flash (adds album tap-to-play)
5. Run Polish phase → Final testing → Flash release

### Files Modified Summary

| File | Tasks | Changes |
|------|-------|---------|
| `esphome/packages/browse.yaml` | T001-T006, T013, T028 | Globals, scripts, API handling |
| `esphome/ui/browse_unified.yaml` | T007-T009, T011, T014-T016, T018 | UI tap handlers, header, button |
| `esphome/packages/navigation.yaml` | T010 | Back navigation handling |

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each phase or logical group
- Stop at any checkpoint to validate story independently
- Avoid: modifying same file section in parallel tasks
- Constitution compliance: All changes must follow §1.2 (responsiveness), §3.1 (modularization), §6.3 (naming)
