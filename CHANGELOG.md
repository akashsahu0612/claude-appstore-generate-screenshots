# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-13

### Added
- Initial release of the `claude-appstore-generate-screenshots` Claude Code skill
- 9-phase automated iOS App Store screenshot capture pipeline
- 5 automation strategies (A–E) covering Flutter and native iOS projects
- Flutter `integration_test` strategy with `binding.takeScreenshot()` for deterministic captures
- Native iOS strategies via fastlane snapshot, deep links (`xcrun simctl openurl`), and new UITest target
- Automatic project type detection (Flutter vs native iOS, fvm detection)
- Screen discovery via route scanning (go_router, Navigator.pushNamed, auto_route, SwiftUI NavigationLink, storyboard)
- Login/auth flow detection with session-only credential handling
- Automatic strategy selection based on detected project structure
- Code generation from templates with per-project variable substitution
- Pre-flight checks (simulator availability, build validation, output directory)
- Clean status bar injection (9:41, full WiFi, full battery) via `xcrun simctl status_bar`
- Screenshot quality assessment (Great / Usable / Retake ratings)
- Memory persistence via `aso_screenshots_auto.md` for session resumption
- Templates for Flutter (`flutter_screenshot_test.dart.template`), native iOS (`ios_snapshot_test.swift.template`), and fastlane (`Snapfile.template`)
