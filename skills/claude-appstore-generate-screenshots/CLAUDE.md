# CLAUDE.md

This file provides guidance to Claude Code when working with the `claude-appstore-generate-screenshots` skill.

## What This Is

A Claude Code skill that automates iOS App Store screenshot capture end-to-end. It is invoked via `/claude-appstore-generate-screenshots` from within a user's iOS or Flutter app project — booting the simulator, building the app, navigating to key screens, and capturing clean, status-bar-corrected screenshots.

## Architecture

```
claude-appstore-generate-screenshots/
├── SKILL.md                                      — Main 9-phase skill prompt
├── CLAUDE.md                                     — This file
└── templates/
    ├── flutter_screenshot_test.dart.template     — Flutter integration_test boilerplate
    ├── ios_snapshot_test.swift.template          — Native iOS XCUITest boilerplate
    └── Snapfile.template                         — fastlane snapshot configuration
```

## Phases Overview

| Phase | Name | User Input Required |
|-------|------|---------------------|
| 0 | RECALL | None (presents status, offers resume/restart) |
| 1 | PROJECT DETECTION | None (fully automated) |
| 2 | SCREEN DISCOVERY | Yes — confirm screen list, mode, device |
| 3 | LOGIN DETECTION | Conditional — only if auth flow detected |
| 4 | STRATEGY SELECTION | None (fully automated) |
| 5 | CODE GENERATION | Yes — review generated test code |
| 6 | PRE-FLIGHT CHECKS | Yes — final go/no-go |
| 7 | EXECUTION | None (runs automation end-to-end) |
| 8 | ASSESSMENT & HANDOFF | None (Claude reviews screenshots) |
| 9 | MEMORY PERSISTENCE | None (auto-saves) |

## Strategy Decision Tree

```
Flutter app?
  └── Yes → existing integration_test/?
        ├── Yes → Strategy A (extend existing tests)
        └── No  → Strategy B (new test file)
  └── No (native iOS) → existing UITest target?
        ├── Yes → Strategy C (fastlane snapshot)
        └── No  → deep links detected?
              ├── Yes → Strategy D (xcrun simctl openurl)
              └── No  → Strategy E (needs Xcode UITest target setup)
```

## Template Usage

Templates are loaded from the `templates/` directory relative to the skill. Variables use `{{VARIABLE_NAME}}` syntax and are substituted during Phase 5 code generation:

**Flutter template variables:**
- `{{APP_PACKAGE}}` — from `pubspec.yaml` name field
- `{{LOGIN_BLOCK}}` — empty string if no auth, full login sequence if auth detected
- `{{SCREEN_STEPS}}` — generated per-screen navigation + screenshot calls
- `{{SCREENSHOT_DIR}}` — output directory path (default: `./aso-screenshots`)

**iOS template variables:**
- `{{APP_TARGET}}` — main app target name from xcodeproj
- `{{APP_SCHEME}}` — Xcode scheme name
- `{{LOGIN_BLOCK}}` — empty if no auth, UITest login sequence if auth detected
- `{{SCREEN_STEPS}}` — per-screen navigation + snapshot calls
- `{{SNAPSHOT_DIR}}` — output directory path

## Memory Schema

Memory file: `aso_screenshots_auto.md`

Fields saved (Phase 9):
```yaml
project_type: flutter | ios
fvm: true | false
strategy: A | B | C | D | E
simulator_name: "iPhone 17 Pro"
simulator_udid: "ABC-123..."
screens:
  - name: HomeScreen
    route: /home
    selector: find.byType(HomeScreen)
    screenshot_file: 01-home.png
    rating: Great | Usable | Retake
    notes: "..."
login_required: true | false
# credentials are NEVER saved
output_directory: /path/to/project/aso-screenshots
last_run: 2025-03-15T14:30:00Z
```

## Key Design Decisions

1. **Flutter integration_test > fastlane snapshot for Flutter** — native widget tree access via `binding.takeScreenshot()`, synchronous with pump cycle, no accessibility tree fragility
2. **`binding.takeScreenshot()` > `xcrun simctl screenshot`** — deterministic timing (synchronous with widget pump), no race conditions
3. **`xcrun simctl status_bar override`** — clean 9:41 status bar set before capture, always restored after (even on error)
4. **Credentials as session env vars only** — `--dart-define=TEST_EMAIL=$TEST_EMAIL` for Flutter, `ProcessInfo.processInfo.environment["TEST_EMAIL"]` for iOS; never in memory, never on disk
5. **One question block per phase** — phases 2, 3, 5, 6 ask exactly one focused question; all other phases are silent
6. **Stop on failure** — no silent retries or workarounds; clear error messages with actionable guidance

