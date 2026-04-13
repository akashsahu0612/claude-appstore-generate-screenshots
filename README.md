# claude-appstore-generate-screenshots

A [Claude Code](https://claude.ai/code) skill that automates iOS App Store screenshot capture from end to end — booting the simulator, building your app, navigating to key screens, and capturing clean, status-bar-corrected screenshots ready for ASO.

Works with both **Flutter** and **native iOS** projects.

---

## What It Does

Invoke `/claude-appstore-generate-screenshots` from within your iOS or Flutter project and Claude will:

1. **Detect** your project type (Flutter / native iOS, fvm, existing test infrastructure)
2. **Discover** your key screens via route scanning and offer a confirmation step
3. **Detect** login flows and handle test credentials safely (session-only, never persisted)
4. **Select** the best automation strategy automatically
5. **Generate** a test file from template, pre-filled for your app — you review before anything runs
6. **Pre-flight check** — confirms simulator availability and build validity, then asks for your go-ahead
7. **Execute** — boots simulator, sets clean status bar (9:41), runs automation, captures screenshots
8. **Assess** each screenshot (Great / Usable / Retake) and hands off to `aso-appstore-screenshots`
9. **Persist** session state to memory so you can resume exactly where you left off

**Minimum questions.** Phases 0, 1, 4, 7, 8, and 9 require zero user input. You only answer three focused question blocks: confirm screens, review generated code, and give a final go/no-go.

---

## Automation Strategies

| Strategy | When Selected | Mechanism |
|----------|--------------|-----------|
| **A** | Flutter + existing `integration_test/` | Adds `aso_screenshots_test.dart` alongside existing tests |
| **B** | Flutter, no existing integration tests | Creates new `integration_test/aso_screenshots_test.dart` + `test_driver/` |
| **C** | Native iOS + existing UITest target | Adds snapshot test to existing target via fastlane snapshot |
| **D** | Native iOS + deep link support detected | `xcrun simctl openurl` navigation — no test code required |
| **E** | Native iOS, no UITest target, no deep links | fastlane snapshot + new UITest target (one-time Xcode setup guided) |

Flutter strategies use `binding.takeScreenshot()` — synchronous with the widget pump cycle — for deterministic, race-condition-free captures.

---

## Requirements

- **macOS** with Xcode installed
- **iOS Simulator** (iPhone 16 Pro or newer recommended)
- **Claude Code** CLI (`claude`)
- **Flutter / fvm** — required for Strategy A/B (auto-detected)
- **fastlane** — required for Strategy C/E (optional for A/B/D)

---

## Installation

### Option 1 — Clone and copy (recommended)

```bash
git clone https://github.com/akashsahu0612/claude-appstore-generate-screenshots.git
cp -r claude-appstore-generate-screenshots/skills/claude-appstore-generate-screenshots ~/.claude/skills/
```

> The **entire folder** must be copied — not just `SKILL.md`. It includes `CLAUDE.md` and the `templates/` directory used during code generation.

### Option 2 — Manual download

Download the `skills/claude-appstore-generate-screenshots/` folder and place it at:

```
~/.claude/skills/claude-appstore-generate-screenshots/
```

---

## Usage

Navigate to your iOS or Flutter project root, then invoke the skill in Claude Code:

```
/claude-appstore-generate-screenshots
```

Claude will guide you through the 9-phase pipeline. At minimum you'll answer:
1. **Phase 2** — confirm the discovered screens, choose light/dark mode and device
2. **Phase 5** — review the generated test code before it's written to disk
3. **Phase 6** — final go/no-go before automation runs

---

## Skill Structure

```
skills/claude-appstore-generate-screenshots/
├── SKILL.md                                    — 9-phase skill prompt
├── CLAUDE.md                                   — Architecture guidance for Claude Code
└── templates/
    ├── flutter_screenshot_test.dart.template   — Flutter integration_test boilerplate
    ├── ios_snapshot_test.swift.template        — Native iOS XCUITest boilerplate
    └── Snapfile.template                       — fastlane snapshot configuration
```

---

## Companion Skill: aso-appstore-screenshots

Once screenshots are captured and assessed, Claude hands off to [`aso-appstore-screenshots`](https://github.com/akashsahu0612/claude-aso-appstore-screenshots) which:
- Analyzes your app's core benefits
- Pairs each screenshot with a headline and subheadline
- Generates production-ready App Store screenshots with design overlays

The handoff is seamless — if you invoke `aso-appstore-screenshots` first, it offers to auto-capture screenshots via this skill before continuing to generation.

---

## Contributing

Issues and PRs are welcome. When filing a bug, please include:
- Project type (Flutter / native iOS)
- Strategy selected (A/B/C/D/E)
- Phase where it failed
- Simulator model and iOS version

---

## License

MIT — see [LICENSE](LICENSE)
