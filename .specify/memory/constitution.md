<!--
Sync Impact Report
==================
Version change: 2.1.0 → 2.2.0 (new section added)
Modified principles: None
Added sections:
  - 3. Code Quality Standards (new) - modularization, clean code, documentation
Renumbered sections:
  - Documentation Separation of Concerns: 3 → 4
  - Repository and File Conventions: 4 → 5
  - Naming Conventions: 5 → 6
  - Display and Touch Rules: 6 → 7
  - Backlight, Idle Behavior: 7 → 8
  - Music Assistant UX Rules: 8 → 9
  - Logging and Diagnostics: 9 → 10
  - Quality Gates: 10 → 11
  - Conflict Resolution Rule: 11 → 12
  - PR Review Checklist: 12 → 13
Removed sections: None
Templates requiring updates:
  - .specify/templates/plan-template.md: ✅ Compatible
  - .specify/templates/spec-template.md: ✅ Compatible
  - .specify/templates/tasks-template.md: ✅ Compatible
Follow-up TODOs: None
-->

# ESPHome CYD Music Assistant Controller Constitution

**Project**: Music Assistant Remote UI
**Hardware**: ESP32-2432S028R (Cheap Yellow Display)
**Display**: 240×320 ILI9341 TFT @ Landscape (320×240)
**Touch**: XPT2046 Resistive
**Integration**: Home Assistant Native API → Music Assistant

---

## 1. Core Non-Negotiable Principles

### 1.1 Reliability & Recovery

1. The device MUST boot to a usable UI within **10 seconds**.

2. The device MUST recover gracefully from:
   - Wi-Fi loss
   - Home Assistant API loss
   - Music Assistant unavailability
   - Reboots and OTA updates

3. Missing or unavailable entities MUST NOT crash, reboot, or freeze the UI.

4. The UI MUST always display a valid state from this set:
   - `Boot` — startup in progress
   - `Connecting` — establishing Wi-Fi or HA API connection
   - `Ready` — normal operation
   - `Offline` — network or HA unavailable
   - `Error` — unrecoverable fault with user-visible message

5. State transitions MUST be logged at INFO level.

### 1.2 UI Responsiveness

1. Touch input MUST remain responsive under all operating conditions.

2. The main loop MUST NOT be blocked by:
   - Network waits
   - Home Assistant service calls
   - Long rendering operations

3. UI updates MUST be selective; frequent full-screen redraws are **forbidden**.

4. Touch-to-visual-feedback latency MUST be under **100 ms**.

5. Dynamic elements (progress bar, time) MUST be rate-limited to **2–4 Hz**.

### 1.3 Deterministic Behavior

1. All entity IDs MUST be explicit — no wildcards or pattern matching.

2. All LVGL object IDs MUST be explicit and follow naming conventions (§6).

3. All substitutions MUST be declared in the main YAML file.

4. UI state MUST be modeled as a finite state machine with documented transitions.

5. Behavior MUST be identical across reboots given the same external state.

### 1.4 Safety & No-Surprises

1. All physical outputs (backlight, buzzer) MUST have safe defaults at boot:
   - Backlight: conservative brightness (≤50%)
   - Buzzer: silent

2. Backlight MUST restore the last user-set value when available, after boot completes.

3. No output MUST activate unexpectedly during boot or error states.

4. User-facing controls MUST require intentional interaction (no accidental triggers).

### 1.5 Observability

1. The device MUST expose the following diagnostics as Home Assistant sensors:
   - `uptime`
   - `wifi_rssi`
   - `ip_address`
   - `ha_api_connected` (binary)
   - `free_heap`
   - `reset_reason`

2. Logs MUST use consistent tags:
   - `wifi` — network events
   - `ha` — Home Assistant API events
   - `ui` — UI state changes
   - `touch` — touch input events
   - `media` — Music Assistant / media_player events
   - `perf` — performance metrics

3. Log severity MUST follow:
   - `INFO` — state changes, successful commands
   - `WARN` — transient failures, retries
   - `ERROR` — persistent failures, degraded operation

### 1.6 Security & Privacy

1. Secrets MUST be stored in `secrets.yaml`.

2. `secrets.yaml` MUST be listed in `.gitignore` and MUST NOT be committed.

3. The UI MUST NOT display tokens, API keys, URLs with credentials, or passwords.

4. Home Assistant Native API MUST be used; insecure HTTP endpoints are **forbidden**.

5. OTA updates MUST require a password defined in secrets.

### 1.7 Change Management

1. Every code change MUST map to a specification section or documented requirement.

2. Entity names and IDs MUST remain stable; changes require a documented migration.

3. Every PR MUST include a completed test checklist (§13).

4. Breaking changes MUST be flagged in the commit message with `BREAKING:` prefix.

---

## 2. Architecture Boundaries

### 2.1 YAML vs Custom Code

1. ESPHome YAML MUST be the primary implementation language.

2. All UI MUST be implemented using LVGL via ESPHome's native integration.

3. Custom C++ MAY be used **only** when:
   - ESPHome or LVGL cannot support a requirement, **or**
   - Performance constraints cannot be met otherwise

4. Custom C++ components MUST:
   - Be isolated in `components/<name>/`
   - Include a README documenting:
     - Purpose and justification
     - Failure modes
     - Testing strategy
   - Follow ESPHome component lifecycle (`setup()`, `loop()`, `dump_config()`)
   - Use ESPHome logging macros (`ESP_LOGD`, `ESP_LOGI`, `ESP_LOGW`, `ESP_LOGE`)

### 2.2 Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                     Home Assistant                          │
│  (media_player entities, services, state)                   │
└─────────────────────┬───────────────────────────────────────┘
                      │ Native API
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   Adapter Layer                             │
│  - Translates HA state → UI model                           │
│  - Translates UI events → HA service calls                  │
│  - Handles connection state                                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   LVGL Presentation                         │
│  - Renders UI model                                         │
│  - Captures touch events                                    │
│  - Manages screen transitions                               │
└─────────────────────────────────────────────────────────────┘
```

1. Home Assistant entities and services are the **control plane**.

2. LVGL is the **presentation layer**.

3. An adapter layer MUST mediate between HA state and UI model.

4. Low-level touch handlers MUST NOT call HA services directly.

5. UI events MUST be processed through the adapter layer.

---

## 3. Code Quality Standards

### 3.1 Modularization

1. Each YAML package MUST have a **single responsibility**:
   - `wifi.yaml` — network configuration only
   - `display.yaml` — display hardware setup only
   - `touch.yaml` — touch controller configuration only
   - `media.yaml` — Music Assistant integration only

2. Packages MUST be **loosely coupled**:
   - Packages MUST NOT directly reference internal IDs from other packages
   - Inter-package communication MUST use global variables or ESPHome's built-in mechanisms
   - Removing a non-essential package MUST NOT break compilation

3. **No circular dependencies** between packages.

4. UI screens MUST be self-contained:
   - Each screen file (`now_playing.yaml`, `menu.yaml`) MUST define all its own widgets
   - Shared UI components MUST be extracted to `ui/common.yaml` or similar
   - Screen files MUST NOT depend on widget IDs from other screens

5. Package size limits:
   - Individual YAML files SHOULD NOT exceed **300 lines**
   - Files exceeding this SHOULD be split into logical sub-packages

### 3.2 Clean Code Principles

1. **Meaningful Names**:
   - All IDs MUST clearly describe their purpose
   - Avoid abbreviations except well-known ones (btn, lbl, img)
   - Names MUST be self-documenting

2. **No Magic Numbers**:
   - All numeric constants MUST be defined as substitutions or named variables
   - Examples: timeouts, dimensions, thresholds, pin numbers
   ```yaml
   # BAD
   - delay: 5000ms

   # GOOD
   substitutions:
     idle_dim_timeout: "60s"
   ```

3. **No Magic Strings**:
   - Repeated strings MUST be substitutions
   - Entity IDs, icon names, and color codes MUST be centralized

4. **DRY (Don't Repeat Yourself)**:
   - Repeated configuration blocks MUST be extracted to packages or anchors
   - Similar widgets MUST use YAML anchors or ESPHome's `!include` with variables
   - Three or more repetitions triggers mandatory refactoring

5. **Single Level of Abstraction**:
   - Each lambda or script SHOULD do one thing
   - Complex logic MUST be broken into named scripts
   - Deeply nested conditionals MUST be refactored

6. **Consistent Formatting**:
   - 2-space indentation for all YAML
   - Blank line between logical sections
   - Lists aligned consistently
   - Maximum line length: 100 characters (SHOULD)

7. **Fail Fast**:
   - Validate inputs at boundaries (on_value triggers, API responses)
   - Invalid states MUST be caught and logged immediately
   - MUST NOT silently ignore errors

### 3.3 Code Documentation

1. **File Headers** — Every YAML package MUST start with a comment block:
   ```yaml
   # =============================================================================
   # Package: Display Configuration
   # Purpose: ILI9341 TFT display initialization and LVGL setup
   # Dependencies: None (hardware-level package)
   # Exposes: display component 'main_display'
   # =============================================================================
   ```

2. **Complex Logic** — Lambdas and scripts with non-obvious logic MUST have comments:
   - Explain WHY, not WHAT
   - Document edge cases and assumptions
   - Reference relevant constitution sections if applicable

3. **Substitution Documentation**:
   ```yaml
   substitutions:
     # Device identification
     device_name: "cyd_music_remote"      # ESPHome device name (no spaces)
     friendly_name: "Music Remote"         # Human-readable name for HA

     # Display settings
     screen_rotation: "90"                 # 0=portrait, 90=landscape, etc.
     default_brightness: "50"              # Boot brightness (0-100%)
   ```

4. **Non-Obvious Configuration** — Any configuration that isn't self-evident MUST be commented:
   - Pin assignments with physical location reference
   - Calibration values with how they were derived
   - Workarounds with links to issues or documentation

5. **TODO/FIXME Standards**:
   - `# TODO(category): description` — planned improvement
   - `# FIXME(category): description` — known issue requiring fix
   - `# HACK(category): description` — temporary workaround (MUST include issue link)
   - Categories: `ui`, `perf`, `reliability`, `security`

### 3.4 Custom C++ Documentation (If Applicable)

1. Every custom component MUST include a `README.md` with:
   - **Purpose**: Why this component exists (what ESPHome/LVGL couldn't do)
   - **Interface**: Public methods, configuration options, exposed entities
   - **Dependencies**: Required libraries, ESPHome version constraints
   - **Failure Modes**: What happens when things go wrong
   - **Testing**: How to verify the component works correctly
   - **Maintenance**: Who owns it, how to modify it safely

2. C++ code MUST include:
   - Doxygen-style comments for public methods
   - Inline comments for complex algorithms
   - Clear error messages in log statements

### 3.5 Changelog Maintenance

1. A `CHANGELOG.md` MUST be maintained in the repository root.

2. Format MUST follow [Keep a Changelog](https://keepachangelog.com/):
   ```markdown
   ## [Unreleased]
   ### Added
   ### Changed
   ### Fixed
   ### Removed
   ```

3. Every user-visible change MUST be documented in the changelog.

4. Breaking changes MUST be clearly marked with `**BREAKING**` prefix.

---

## 4. Documentation Separation of Concerns

Documentation follows strict separation between WHAT/WHY (`spec.md`) and HOW (`plan.md`).

### 4.1 spec.md — Product Perspective (What & Why)

**Purpose**: Define what the system should do and why it matters.

**Target Audience**: Product Owner, Stakeholders, Domain Experts

1. spec.md MUST remain **technology-agnostic**.

2. spec.md MUST NOT contain:
   - Implementation details (frameworks, libraries, architecture patterns)
   - Technical terminology (except domain-specific terms)
   - Code snippets or configuration examples
   - References to specific files or modules

3. spec.md MUST contain:
   - User Stories with acceptance criteria
   - Functional Requirements (FR-XXX format)
   - Success Criteria (measurable outcomes)
   - Edge cases and error scenarios (from user perspective)

4. The guiding question: *"What should the system do and why?"*

### 4.2 plan.md — Engineering Perspective (How)

**Purpose**: Define how to implement the requirements from spec.md.

**Target Audience**: Developers, Tech Leads, Code Reviewers

1. plan.md MUST contain ALL technical details:
   - Language, frameworks, and library choices
   - Architecture patterns and design decisions
   - File structure and module organization
   - Technical Context (dependencies, storage, testing approach)

2. plan.md MUST include:
   - Constitution Checks (verification against this document)
   - Complexity Tracking (justification for deviations)
   - Technical risk assessment

3. The guiding question: *"How do we implement the requirements from spec.md?"*

### 4.3 Violations & Enforcement

1. **Technical details in spec.md are a merge blocker.**

2. spec.md reviews MUST verify technology-agnosticism:
   - No framework names (ESPHome, LVGL, etc.)
   - No file paths or code references
   - No architecture terminology (adapter layer, component lifecycle, etc.)

3. All "HOW" discussions MUST be documented in:
   - plan.md (design decisions)
   - Code comments (implementation details)
   - NOT in spec.md

4. Constitution Checks in plan.md MUST validate this separation.

### 4.4 Rationale

Clear separation provides:
- **Maintainability**: spec.md remains valid even when tech stack changes
- **Focus**: Product discussions center on user value, not implementation
- **Clarity**: Each document has a single purpose and audience
- **Reviewability**: Technical and product concerns reviewed separately

---

## 5. Repository and File Conventions

### 5.1 Directory Structure

```
esphome_cyd/
├── esphome/
│   ├── main.yaml                 # Entrypoint (includes packages)
│   ├── secrets.yaml              # Credentials (gitignored)
│   ├── packages/
│   │   ├── wifi.yaml             # Wi-Fi configuration
│   │   ├── display.yaml          # ILI9341 display setup
│   │   ├── touch.yaml            # XPT2046 touch configuration
│   │   ├── backlight.yaml        # PWM backlight control
│   │   ├── media.yaml            # Music Assistant integration
│   │   └── diagnostics.yaml      # Sensors and debug info
│   └── ui/
│       ├── now_playing.yaml      # Now Playing screen
│       ├── menu.yaml             # Menu / shortcuts screen
│       └── overlays.yaml         # Popups, toasts, status bars
├── components/                   # Custom ESPHome components (if any)
│   └── <name>/
│       ├── __init__.py
│       ├── <name>.h
│       ├── <name>.cpp
│       └── README.md             # Required: justification and docs
├── fonts/                        # LVGL fonts
└── docs/
    ├── wiring.md                 # CYD variant wiring notes
    ├── touch_calibration.md      # Calibration procedure
    └── known_issues.md           # Issues and recovery steps
```

### 5.2 File Rules

1. `main.yaml` MUST be the single entrypoint.

2. All reusable configuration MUST be in `packages/`.

3. All UI definitions MUST be in `ui/`.

4. Each custom component MUST have its own directory with a README.

5. Documentation MUST be maintained in `docs/`.

---

## 6. Naming Conventions

### 6.1 General Rules

1. All IDs MUST use `lower_snake_case`.

2. Entity names MUST be human-readable and stable across updates.

3. Names MUST NOT include version numbers or temporary identifiers.

### 6.2 Substitutions (Required)

The following substitutions MUST be defined in `main.yaml`:

| Substitution | Purpose | Example |
|--------------|---------|---------|
| `device_name` | ESPHome device name | `cyd_music_remote` |
| `friendly_name` | Human-readable name | `Music Remote` |
| `screen_rotation` | Display rotation | `90` (landscape) |
| `default_brightness` | Boot brightness (0-100) | `50` |

### 6.3 LVGL Object IDs

LVGL object IDs MUST follow this pattern:

```
ui_<page>_<component>_<role>
```

| Part | Description | Examples |
|------|-------------|----------|
| `ui_` | Prefix (always) | — |
| `<page>` | Screen name | `nowplaying`, `menu`, `overlay` |
| `<component>` | Widget type | `btn`, `lbl`, `bar`, `img`, `slider` |
| `<role>` | Specific function | `play`, `title`, `progress`, `volume` |

**Examples:**
- `ui_nowplaying_btn_play`
- `ui_nowplaying_lbl_title`
- `ui_nowplaying_bar_progress`
- `ui_menu_btn_player1`
- `ui_overlay_lbl_error`

### 6.4 Entity IDs

Entity IDs MUST follow:

```
<domain>.<device_name>_<function>
```

**Examples:**
- `sensor.cyd_music_remote_uptime`
- `sensor.cyd_music_remote_wifi_rssi`
- `switch.cyd_music_remote_debug_mode`
- `number.cyd_music_remote_brightness`

---

## 7. Display and Touch Rules (CYD-Specific)

### 7.1 Display Performance

1. The UI MUST assume limited RAM (~320KB) and SPI bandwidth.

2. Fonts MUST be limited to:
   - **One text font** (with required sizes)
   - **One icon font** (Material Design Icons or similar)

3. Frequent updates (progress bar, elapsed time) MUST be rate-limited to **2–4 Hz**.

4. Full-screen invalidation MUST NOT be used for dynamic elements.

5. Partial invalidation MUST be used for all updates.

6. Animations MUST be:
   - Minimal (short duration, simple transitions)
   - Automatically disabled in offline or low-performance states

7. Image assets MUST be:
   - Pre-converted to LVGL format
   - Sized appropriately (no runtime scaling of large images)

### 7.2 Resistive Touch Rules

1. Touch input MUST be treated as inherently noisy.

2. The following MUST be implemented:
   - Debouncing (ignore rapid repeated touches)
   - Micro-movement filtering (ignore jitter)
   - Optional smoothing for drag operations

3. Touch calibration MUST be:
   - Supported via a calibration screen or procedure
   - Persisted across reboots (flash or HA entity)

4. Axis configuration MUST be adjustable:
   - X/Y swap
   - X invert
   - Y invert

5. Touch feedback MUST be provided within **100 ms**:
   - Visual state change (button press effect)
   - Optional haptic/audio feedback if buzzer available

6. Touch targets MUST be minimum **44×44 pixels**.

---

## 8. Backlight, Idle Behavior, and Burn-in Mitigation

### 8.1 Backlight Control

1. Backlight MUST be controllable via:
   - On-device UI (brightness slider or buttons)
   - Home Assistant entity (`number.<device>_brightness`)

2. Backlight brightness MUST be persisted and restored on boot.

3. Boot brightness MUST be conservative (≤50%) until user preference loads.

### 8.2 Idle Behavior

1. Idle detection MUST track time since last touch.

2. Default idle behavior (configurable via substitutions or HA):

| Condition | Action | Default |
|-----------|--------|---------|
| Idle > 60 seconds | Dim backlight to 20% | Enabled |
| Idle > 5 minutes | Turn off backlight | Enabled |
| Any touch | Wake immediately, restore brightness | Always |

3. Idle timeouts MUST be configurable via Home Assistant entities.

4. Media playback state MAY influence idle behavior (e.g., stay awake while playing).

### 8.3 Burn-in Mitigation

1. The Now Playing screen MUST NOT have permanently static high-contrast elements.

2. Recommended mitigations:
   - Subtle element position shifting over time
   - Progress bar animation prevents static pixels
   - Screen-off on extended idle

3. Debug/diagnostic overlays MUST auto-hide or be time-limited.

---

## 9. Music Assistant UX Rules

### 9.1 Required Screens

#### Now Playing (Default Screen)

The Now Playing screen MUST display:

| Element | Required | Notes |
|---------|----------|-------|
| Track title | YES | Scrolling if truncated |
| Artist | YES | — |
| Album | SHOULD | May be hidden if space constrained |
| Album art | SHOULD | Placeholder if unavailable |
| Progress bar | YES | Shows elapsed/total time |
| Play/Pause button | YES | State-aware icon |
| Previous track | YES | — |
| Next track | YES | — |
| Volume slider | YES | — |
| Connection status | YES | Icon or indicator |

#### Menu / Shortcuts Screen

The Menu screen MUST provide:

| Element | Required | Notes |
|---------|----------|-------|
| Player selection | YES (if multiple) | Switch between media_players |
| Favorites/Presets | SHOULD | Top N items |
| Queue controls | MAY | Optional, space permitting |
| Settings access | SHOULD | Brightness, calibration |

### 9.2 Offline Behavior

1. When Home Assistant is unavailable:
   - UI MUST display `Offline` state clearly
   - All media controls MUST be disabled or visually dimmed
   - Touch on disabled controls MUST be safely ignored (no queued actions)

2. When Music Assistant is unavailable but HA is connected:
   - UI MUST indicate media service unavailable
   - Device diagnostics MUST remain functional

3. The UI MUST NOT repeatedly retry failed service calls.

4. Reconnection MUST be handled by the adapter layer with exponential backoff.

### 9.3 Feedback and Errors

1. All user actions MUST provide immediate UI feedback:
   - Button press: visual state change within 100 ms
   - Service call initiated: loading indicator if >500 ms expected

2. Optimistic UI updates are permitted:
   - Update UI immediately on user action
   - Reconcile with actual state within **2 seconds**
   - Revert if service call fails

3. Error messages MUST be:
   - Concise (fits on screen)
   - User-friendly (no technical jargon)
   - Actionable when possible ("Check network connection")

4. Errors MUST auto-dismiss after **5 seconds** or on touch.

---

## 10. Logging and Diagnostics

### 10.1 Log Standards

1. All log messages MUST include a tag from the approved list (§1.5 Observability).

2. Log levels MUST be used correctly:

| Level | Use Case |
|-------|----------|
| `DEBUG` | Verbose info, disabled by default |
| `INFO` | State changes, successful operations |
| `WARN` | Transient failures, retries, degraded states |
| `ERROR` | Persistent failures, requires attention |

3. Logs MUST NOT contain:
   - Secrets or credentials
   - Full API responses (use summaries)
   - High-frequency spam (>10 messages/second sustained)

### 10.2 Debug Mode

1. A "Debug Mode" switch MUST exist (`switch.<device>_debug_mode`).

2. When Debug Mode is enabled:
   - Log verbosity increases to DEBUG level
   - A diagnostics overlay MAY be shown (FPS, heap, connection state)

3. Debug Mode MUST NOT affect normal operation or reliability.

4. Debug Mode SHOULD auto-disable after 30 minutes.

---

## 11. Quality Gates (Definition of Done)

Every change MUST pass all applicable gates before merge:

### 11.1 Build Gates

| Gate | Requirement |
|------|-------------|
| YAML Lint | All YAML files pass `yamllint` |
| ESPHome Compile | `esphome compile main.yaml` succeeds without errors |
| No Warnings | Compile produces no warnings (or warnings are documented exceptions) |

### 11.2 Functional Gates

| Gate | Requirement |
|------|-------------|
| Cold Boot | UI visible and interactive within 10 seconds |
| Wi-Fi Recovery | Disconnect Wi-Fi → UI shows Offline → Reconnect → UI shows Ready within 30s |
| HA API Recovery | Stop HA → UI shows Offline → Start HA → UI shows Ready within 30s |
| Media Unavailable | media_player unavailable → UI shows appropriate state → Available → UI recovers |
| Touch Response | Touch input responds within 100ms under normal operation |
| No UI Stutter | No visible stutter during progress bar updates or screen transitions |

### 11.3 Stability Gates

| Gate | Requirement |
|------|-------------|
| Heap Stability | Free heap does not decrease over 1 hour of operation |
| No Crashes | 24-hour soak test without crash or reboot |
| OTA Update | OTA update completes successfully, device boots correctly |

### 11.4 Security Gates

| Gate | Requirement |
|------|-------------|
| No Committed Secrets | `secrets.yaml` not in repository |
| No Exposed Credentials | UI does not display any secrets |

---

## 12. Conflict Resolution Rule

When constitutional principles conflict, resolve in this priority order:

| Priority | Principle | Rationale |
|----------|-----------|-----------|
| 1 | Safety and Reliability | Device must not cause harm or become unusable |
| 2 | UI Responsiveness | A frozen UI is effectively broken |
| 3 | Security and Privacy | Credentials must never be exposed |
| 4 | Maintainability | Code must remain understandable |
| 5 | Feature Completeness | Features are secondary to stability |

**Example:** If adding a feature would compromise boot time reliability, the feature MUST be deferred or redesigned.

---

## 13. PR Review Checklist

Every PR MUST include this checklist with all applicable items checked:

```markdown
## PR Review Checklist

### Build
- [ ] YAML lint passes
- [ ] ESPHome compile succeeds
- [ ] No new warnings (or exceptions documented)

### Functional Testing
- [ ] Cold boot completes in <10 seconds
- [ ] Wi-Fi disconnect/reconnect recovers gracefully
- [ ] HA API disconnect/reconnect recovers gracefully
- [ ] Media unavailable state handled correctly
- [ ] Touch remains responsive
- [ ] No visible UI stutter

### Stability
- [ ] Heap usage stable (no sustained decrease over 15+ minutes)
- [ ] No crashes during testing

### Security
- [ ] No secrets in committed files
- [ ] No credentials displayed in UI

### Documentation
- [ ] Change maps to spec section: ________________
- [ ] Breaking changes flagged (if applicable)
- [ ] Entity ID changes documented (if applicable)
- [ ] spec.md contains no technical details (§4)
- [ ] plan.md contains all implementation decisions (§4)

### Code Quality
- [ ] Follows naming conventions (§6)
- [ ] LVGL objects use correct ID pattern
- [ ] Custom C++ has README (if applicable)
- [ ] Logging uses correct tags and levels

### Code Quality Standards (§3)
- [ ] Each package has single responsibility
- [ ] No circular dependencies between packages
- [ ] No magic numbers (use substitutions)
- [ ] No repeated code blocks (DRY)
- [ ] File headers present on all packages
- [ ] Complex logic is commented (WHY not WHAT)
- [ ] CHANGELOG.md updated (if user-visible change)
```

---

## Governance

This constitution defines enforceable standards for the ESPHome CYD Music Assistant Controller project. All contributions MUST comply.

### Amendment Process

1. Propose amendment with clear rationale
2. Demonstrate improvement to safety, reliability, or maintainability
3. Update constitution version:
   - **MAJOR**: Removes or redefines existing rules
   - **MINOR**: Adds new rules or sections
   - **PATCH**: Clarifies existing rules without changing meaning
4. Update dependent templates and documentation

### Compliance

- All PRs MUST pass quality gates (§11)
- Constitutional violations MUST be resolved before merge
- Exceptions require documented justification and approval

### Version History

| Version | Date | Change |
|---------|------|--------|
| 2.2.0 | 2026-02-01 | Added Code Quality Standards (§3): modularization, clean code, documentation |
| 2.1.0 | 2026-02-01 | Added Documentation Separation of Concerns (§4) |
| 2.0.0 | 2026-02-01 | Complete restructure as enforceable rules |
| 1.2.1 | 2026-02-01 | Added LVGL requirement |
| 1.2.0 | 2026-02-01 | Added Design-First principle |
| 1.1.0 | 2026-02-01 | Added Library-First principle |
| 1.0.0 | 2026-02-01 | Initial creation |

**Version**: 2.2.0 | **Ratified**: 2026-02-01 | **Last Amended**: 2026-02-01
