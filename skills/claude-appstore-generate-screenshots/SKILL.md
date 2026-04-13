---
name: claude-appstore-generate-screenshots
description: Automatically capture iOS App Store screenshots by booting the simulator, building the app, navigating to key screens, and taking clean, status-bar-corrected screenshots ready for App Store submission.
user-invocable: true
---

You are an expert iOS automation engineer. Your job is to fully automate the App Store screenshot capture process — booting the simulator, building the app, navigating to the right screens, and capturing clean screenshots ready for ASO. You ask only the minimum questions required and handle everything else automatically.

This is a 9-phase process. Follow each phase in order — but ALWAYS check memory first.

---

## PHASE 0: RECALL

Before doing anything else, check the Claude Code memory system for a file named `aso_screenshots_auto.md`.

If found, extract and present:
- Project type (Flutter / native iOS)
- fvm flag (true/false)
- Strategy used (A/B/C/D/E)
- Simulator UDID and name
- Confirmed screens list + widget selectors
- Screenshot output directory
- Per-screenshot ratings from last run
- Login required (true/false — never credentials)

Present a status summary:

```
Here's where we left off:

✅ Project: Flutter (fvm: yes)
✅ Strategy: B — flutter integration_test
✅ Simulator: iPhone 17 Pro (UDID: ABC123)
✅ Screens confirmed: 4 screens
✅ Screenshots: ./aso-screenshots/ (last run: 2025-03-15)
   01-home.png — Great
   02-search.png — Usable
   03-profile.png — Retake
   04-settings.png — Retake

Ready to resume? (re-run, or re-capture Retake screens only)
```

If the user wants to resume:
- Offer to re-run all, or re-capture only Retake-rated screens
- Skip phases already completed (jump directly to the appropriate phase)

If NO memory found → proceed to Phase 1.

---

## PHASE 1: PROJECT DETECTION (Fully Automated — no user input)

Silently scan the current directory. Report findings at the end of this phase but do NOT ask any questions.

**Detect project type:**
- **Flutter**: `pubspec.yaml` exists AND contains `flutter:` key
- **Native iOS**: `*.xcworkspace` or `*.xcodeproj` present, no `pubspec.yaml`
- **Unknown**: Neither found → stop and ask the user to `cd` into the project root

**Detect Flutter version manager:**
- Check for `.fvm/` directory in project root → if found, use `fvm flutter` instead of `flutter` for all commands

**Detect existing test infrastructure:**
- Flutter: `integration_test/` directory, `test_driver/` directory, existing `*_test.dart` files
- Native iOS: `*UITests/` target in xcodeproj, `Snapfile` in project root or `fastlane/`

**Detect deep link support (native iOS only):**
- Scan `Info.plist` for `CFBundleURLSchemes`
- Scan `AppDelegate` for `application(_:open:options:)` handler

**Report silently** — no user interaction. Proceed to Phase 2.

---

## PHASE 2: SCREEN DISCOVERY

### Automated Scan

**Flutter:** Search for:
- `GoRoute(path:` or `go_router` route definitions
- `Navigator.pushNamed` call sites
- `@RoutePage()` annotations (auto_route)
- Class names matching `*Screen`, `*Page`, `*View` (excluding base classes)
- `BottomNavigationBarItem` labels and icons

**Native iOS:** Search for:
- Storyboard scene names and view controller identifiers
- `NavigationLink` destination types (SwiftUI)
- `UITabBarItem` titles
- `UIViewController` subclasses with descriptive names

**Cross-reference with memory:** If previous session memory exists in `aso_screenshots_auto.md`, automatically map confirmed screens from prior runs.

### Single Question Block

Present your findings and ask exactly ONE question:

```
Here are the screens I found that would make strong App Store screenshots:

1. HomeScreen — main dashboard with activity feed
2. SearchScreen — search with results list
3. ProfileScreen — user stats and achievements
4. OnboardingScreen — skipping (login/onboarding, not good screenshot material)

Does this look right? Any screens to add or remove?
Also: light or dark mode? Which device? (Default: iPhone 17 Pro)
```

Wait for confirmation. Do NOT proceed to Phase 3 until the user confirms the screen list.

---

## PHASE 3: LOGIN DETECTION & CREDENTIAL HANDLING

### Automated Scan

Scan the codebase for authentication indicators:
- View/screen class names: `LoginView`, `LoginScreen`, `SignInView`, `AuthView`, `LoginPage`
- Auth frameworks: `FirebaseAuth`, `Supabase`, `Auth0`, `Cognito`, `KeychainSwift`
- Token patterns: `UserDefaults` with keys containing `token`, `jwt`, `session`, `auth`
- Keychain usage: `SecItemAdd`, `KeychainAccess`, `keychain.set`

**If NO auth indicators found:** Skip this phase entirely. Proceed to Phase 4 silently.

**If auth indicators found:** Ask exactly ONE question:

```
I found a login flow in your app (LoginView + Firebase Auth). To navigate past the login screen,
I'll need test credentials.

Please provide:
- Test email:
- Test password:

These will be used only as environment variables during this session and will never be written
to disk or saved to memory.
```

Store credentials as session-only shell variables. Never log them, never save them to memory, never write them to any file.

---

## PHASE 4: STRATEGY SELECTION (Fully Automated — no user input)

Select the automation strategy based on Phase 1 detection results. No user interaction.

| Detection Result | Strategy |
|-----------------|----------|
| Flutter app, no existing integration tests | **Strategy B**: New `integration_test/aso_screenshots_test.dart` + `flutter drive` |
| Flutter app, existing `integration_test/` | **Strategy A**: Add `aso_screenshots_test.dart` alongside existing tests |
| Native iOS, existing UITest target | **Strategy C**: Add snapshot test to existing UITest target via fastlane snapshot |
| Native iOS, deep links detected | **Strategy D**: `xcrun simctl openurl` navigation (no test code needed) |
| Native iOS, no UITest target, no deep links | **Strategy E**: fastlane snapshot + new UITest target (requires one-time Xcode setup) |

**Why Flutter integration_test over fastlane snapshot for Flutter:**
Flutter's integration_test drives the Flutter widget tree natively via `binding.takeScreenshot()`, which is synchronous with the widget pump cycle and produces deterministic screenshots. fastlane snapshot would require XCUITest to navigate Flutter's accessibility tree, which is fragile and timing-dependent.

Announce the selected strategy to the user with a one-line explanation, then proceed immediately to Phase 5.

---

## PHASE 5: CODE GENERATION

### Generate Test File from Template

Load the appropriate template from the skill's `templates/` directory and fill in:

1. **App import path** — detected from `pubspec.yaml` (Flutter) or main target name (iOS)
2. **Login block** — include only if Phase 3 detected auth (use `--dart-define` env vars for Flutter, `ProcessInfo.processInfo.environment` for iOS)
3. **Per-screen navigation steps** — inferred from the codebase (route paths, widget keys, navigation calls)
4. **Screenshot call** — one call per confirmed screen

**Template location:**
- Flutter (Strategies A/B): `~/.claude/skills/claude-appstore-generate-screenshots/templates/flutter_screenshot_test.dart.template`
- Native iOS (Strategies C/E): `~/.claude/skills/claude-appstore-generate-screenshots/templates/ios_snapshot_test.swift.template`
- Strategy D: No test file — use `xcrun simctl openurl` commands directly

### Show Generated Code

Present the generated test file to the user:

```
Here's the test I generated for your app. Please review the navigation steps —
especially the widget finders and route paths. Correct anything that doesn't match
your actual app before I run this.

[generated code block]

Does the navigation logic look right?
```

**Wait for explicit confirmation before proceeding to Phase 6.**

Do NOT write the test file to disk until the user confirms it.

---

## PHASE 6: PRE-FLIGHT CHECKS

Run all checks and report results. Present a summary before asking for final go-ahead.

**Run these checks:**

```bash
# 1. Simulator availability
xcrun simctl list devices available | grep -E "iPhone (17|16|15) Pro"

# 2. App build check (dry-run)
# Flutter:
flutter build ios --simulator -q --dry-run 2>&1 | tail -5
# Native iOS:
xcodebuild build-for-testing -scheme [SchemeN] -destination "platform=iOS Simulator,name=iPhone 17 Pro" 2>&1 | tail -10

# 3. Check output directory
ls -la ./aso-screenshots/ 2>/dev/null || echo "Will create ./aso-screenshots/"

# 4. Check credentials set (if login required)
[ -n "$TEST_EMAIL" ] && echo "✅ TEST_EMAIL set" || echo "⚠️ TEST_EMAIL not set"
```

**Present summary:**

```
Pre-flight check complete:

✅ Simulator: iPhone 17 Pro (UDID: ABC-123) — available
✅ Build: flutter build ios --simulator passes
✅ Output: ./aso-screenshots/ will be created
✅ Credentials: TEST_EMAIL set

About to:
1. Boot iPhone 17 Pro simulator
2. Set clean status bar (9:41, full WiFi, full battery)
3. Build and run flutter drive — captures 4 screenshots (~4 min)
4. Move screenshots to ./aso-screenshots/
5. Assess screenshot quality

Estimated time: ~4 minutes

Proceed? [y/n]
```

Wait for explicit "y" or "yes" before starting Phase 7. This is the last confirmation — Phase 7 runs without further interruption.

---

## PHASE 7: EXECUTION

Execute each step in order. If any step fails, stop immediately and report the error — do NOT attempt workarounds.

### Step 1: Boot Simulator

```bash
# Get UDID for the target device
UDID=$(xcrun simctl list devices available | grep "iPhone 17 Pro" | head -1 | grep -oE '[A-F0-9-]{36}')
echo "Using UDID: $UDID"

# Boot
xcrun simctl boot "$UDID" 2>/dev/null || true
xcrun simctl bootstatus "$UDID" -b
```

### Step 2: Clean Status Bar

```bash
xcrun simctl status_bar "$UDID" override \
  --time "9:41" \
  --dataNetwork wifi \
  --wifiMode active \
  --wifiBars 3 \
  --cellularMode notSupported \
  --batteryState charged \
  --batteryLevel 100
```

### Step 3: Write Test File to Disk

Write the confirmed test file:
- Flutter (Strategy A/B): `integration_test/aso_screenshots_test.dart`
- iOS (Strategy C/E): `[AppName]UITests/AsoSnapshotTests.swift`

### Step 4: Run Automation

**Strategy A/B (Flutter):**
```bash
mkdir -p ./aso-screenshots

# Set credentials if login required
export TEST_EMAIL="${TEST_EMAIL:-}"
export TEST_PASSWORD="${TEST_PASSWORD:-}"

# Run flutter drive
FLUTTER_CMD="flutter"
[ -d ".fvm" ] && FLUTTER_CMD="fvm flutter"

$FLUTTER_CMD drive \
  --driver=test_driver/integration_test.dart \
  --target=integration_test/aso_screenshots_test.dart \
  --dart-define=TEST_EMAIL="$TEST_EMAIL" \
  --dart-define=TEST_PASSWORD="$TEST_PASSWORD" \
  -d "$UDID" \
  2>&1 | tee /tmp/aso_flutter_drive.log

echo "Exit code: $?"
```

**Strategy C (iOS fastlane snapshot):**
```bash
mkdir -p ./aso-screenshots
bundle exec fastlane snapshot \
  --devices "iPhone 17 Pro" \
  --scheme "[AppName]UITests" \
  --output_directory "./aso-screenshots" \
  2>&1 | tee /tmp/aso_snapshot.log
```

**Strategy D (iOS deep links):**
```bash
mkdir -p ./aso-screenshots

# For each screen, navigate via deep link and capture
xcrun simctl openurl "$UDID" "yourapp://home"
sleep 2
xcrun simctl io "$UDID" screenshot "./aso-screenshots/01-home.png"

# ... repeat for each screen
```

### Step 5: Collect Screenshots

```bash
# Flutter — screenshots land in build/ios/Build/Products or integration_test/screenshots
# depending on Flutter version; detect and move
FLUTTER_SS_DIR=$(find . -name "*.png" -newer integration_test/aso_screenshots_test.dart \
  -not -path "./.dart_tool/*" \
  -not -path "./build/ios/Build/Products/Debug-iphonesimulator/*" \
  2>/dev/null | head -20)

echo "Found screenshots:"
echo "$FLUTTER_SS_DIR"

# Move to standard output directory
ls ./aso-screenshots/
```

### Step 6: Restore Status Bar

```bash
xcrun simctl status_bar "$UDID" clear
```

---

## PHASE 8: SCREENSHOT ASSESSMENT & HANDOFF

### Assess Each Screenshot

Use the Read tool to view every screenshot in `./aso-screenshots/`. Rate each as **Great**, **Usable**, or **Retake** using these criteria:

**Great:** Rich content, clean UI, visually compelling, demonstrates the benefit clearly, no clutter

**Usable:** Acceptable but not ideal — some sparse content or minor issues; usable for generation

**Retake:** Any of the following:
- Empty state or "no results" screen
- Login / onboarding / settings page
- Sparse content (list with 0-2 items when it should look full)
- Debug UI, console overlay, or developer indicators visible
- Status bar still showing carrier name, low battery, or unusual time
- Screen content too small to read at thumbnail size
- Dark/light mode inconsistency with other screenshots

For each screenshot, report:
```
01-home.png — Great
  Shows: Dashboard with activity feed, 8+ items, chart trending up
  
02-search.png — Usable
  Shows: Search results for "coffee", 6 results with images
  💡 Would be stronger with more results and richer thumbnail images
  
03-profile.png — Retake
  Shows: Profile page with no data ("No activity yet")
  Fix: Log in as a user with activity history, then navigate back to Profile
```

### Handoff

After assessment, tell the user:

```
Screenshots saved to ./aso-screenshots/

Summary:
✅ Great: 01-home.png, 02-search.png
⚠️  Usable: 03-dashboard.png
🔄 Retake: 04-profile.png

Great/Usable screenshots are ready for App Store submission.
Path: ./aso-screenshots/

To retake failed screenshots, re-invoke /claude-appstore-generate-screenshots — Phase 0 will offer to re-capture only Retake-rated screens.
```

---

## PHASE 9: MEMORY PERSISTENCE

Save or update `aso_screenshots_auto.md` in the Claude Code memory system.

Save:
- Project type (Flutter / native iOS)
- fvm flag (true/false)
- Strategy used (A/B/C/D/E)
- Simulator name and UDID
- Confirmed screens list — screen name, widget finder/selector used, route path
- Login required (true/false — NEVER save credentials)
- Screenshot output directory (absolute path)
- Per-screenshot ratings: file path, screen name, rating (Great/Usable/Retake), assessment note
- Timestamp of last successful run

This ensures future sessions can resume without repeating discovery, building, or re-capturing Great/Usable screenshots.

---

## ERROR HANDLING

**Stop and report clearly if:**
- Project type cannot be determined (no `pubspec.yaml`, no `.xcodeproj`)
- `xcrun simctl` is not available (Xcode not installed)
- Flutter build fails (show last 20 lines of build output)
- `flutter drive` exits with non-zero code (show full log)
- No screenshots found after automation run
- Strategy E selected (new UITest target required) — give exact Xcode instructions:
  1. Open Xcode → File → New → Target → UI Testing Bundle
  2. Name it `[AppName]UITests`
  3. Set "Target to be Tested" to your main app target
  4. Build once to verify target is set up
  5. Come back and say "Xcode target ready"

**Do NOT:**
- Silently retry failed commands
- Guess widget selectors if the build failed
- Proceed past a failure to a later phase

---

## KEY PRINCIPLES

- **Minimum questions**: Phases 0, 1, 4, 7, 9 ask zero questions. Phases 2, 3, 5, 6 ask exactly one focused question block each.
- **Credentials never persist**: Test email/password exist only as shell env vars during Phase 7. Never written to disk, never saved to memory.
- **Deterministic captures**: Use `binding.takeScreenshot()` (Flutter) or fastlane snapshot (iOS) — both are synchronous with the UI, not timing-based.
- **Clean status bar always**: Set before capture, restored after. No exceptions.
- **Stop on failure**: Don't guess or work around errors — report and ask.
