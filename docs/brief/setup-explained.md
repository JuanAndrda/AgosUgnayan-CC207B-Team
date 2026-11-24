# Setup (PR 1): what changed, why, and how to change it

Owner: Juan. Branch: `feature/setup`. Folder: `B/AgosUgnayan` in your fork. **Not committed or pushed yet.**

## What I added

| File | What it does |
|---|---|
| `lib/main.dart` | Starts the app. Catches every error (`FlutterError.onError` + `runZonedGuarded`) and logs it. Connects to Firebase, and returns false instead of crashing if Firebase isn't set up. On web, centers the phone layout. |
| `lib/app_router.dart` | All screen paths in one place (go_router). There is only one route for now. |
| `lib/screens/setup_check_screen.dart` | **Temporary** screen showing the theme and whether Firebase is connected. Login replaces it in PR 2. |
| `lib/utils/theme.dart` | Colors, sizes, fonts and the Material theme, all from the Figma v3 frames (green `#4A7F2C`, dark green buttons `#2F5E1C`, background `#F1F2ED`, cards with radius 14, inputs with radius 10, and so on). |
| `lib/utils/strings.dart` | All text the user sees, including friendly error and validation messages. |
| `lib/utils/validators.dart` | Required, email, password, matching password, positive number, price (2 decimals) and max length. |
| `lib/utils/app_logger.dart` | Prints only in debug mode, with an `[AgosUgnayan]` prefix. |
| `lib/utils/app_exception.dart` | The one error type services throw, with a safe message, plus `friendlyMessage()`. |
| `lib/utils/snackbars.dart` | `showMessage`, `showError` and `showUndoMessage` for feedback after every action. |
| `lib/widgets/async_state_view.dart` | Shared loading, error (with Try again), empty and data view. |
| `lib/widgets/confirm_dialog.dart` | `showConfirmDialog` (Delete / Cancel) and `showDiscardChangesDialog`. |
| `tool/firebase_options_example.dart` | Fake Firebase settings so the code compiles and CI works without our real keys. |
| `test/` (3 files) | 19 tests for validators, AsyncStateView and the confirm dialog. |
| `.github/workflows/ci.yml` (repo root) | Runs format, analyze and test on GitHub for every push and PR. |
| `README.md`, `CLAUDE.md`, `docs/` | Setup steps, the package list, the checklist, the AI Usage Log and the Topic Coverage Matrix. |
| `.gitignore` | Blocks `lib/firebase_options.dart` and `google-services.json`, so no Firebase keys are ever committed. |

## Why it works this way
- **Screens never talk to Firebase.** Services will, and they turn errors into an `AppException` with friendly text. The screen just shows `AsyncStateView` with that message.
- **One place for each thing.** Color in `theme.dart`, text in `strings.dart`, paths in `app_router.dart`. A live change during the defense is then one line.
- **No keys in git.** Each member makes their own `lib/firebase_options.dart` with `flutterfire configure`.

## How to change it (practice for the defense)
- **Change the main green:** edit `AppColors.primary` in `lib/utils/theme.dart`, then hot restart (`R`).
- **Change an error message:** edit it in `lib/utils/strings.dart`. Tests compare against the same constant, so they still pass.
- **Add a validator:** copy `positiveNumber` in `validators.dart`, rename it, change the check, and add a test in `test/validators_test.dart`.
- **Add a screen:** add a path in `AppRoutes`, then a `GoRoute` in `buildRouter`.

## What I tested
- `dart format .`: done.
- `flutter analyze`: no issues.
- `flutter test`: 19 passed.
- `flutter build web`: built and opened in a browser. The setup screen shows the green app bar and "Firebase not set up yet", which is expected until you run `flutterfire configure`.

## Known gaps in this PR
- **Fonts:** Figma uses Inter and Barlow Condensed. They aren't bundled yet, so the app falls back to the default font.
- **Icons:** Figma's icons are custom SVGs. Showing SVGs needs another package, so screens will use the closest Material icons. That's a small visual difference from Figma.
- **App Check:** comes in PR 5 (AI), where it matters most.

## AI Usage Log line (already in docs/ai-usage-log.md)
| Juan Andrada | 2026-11-01 | Claude Code | Set up the Flutter project: theme from Figma v3, strings, validators, logger, shared widgets, routing, README and CI | lib/utils/*, async_state_view.dart, confirm_dialog.dart, app_router.dart, main.dart, setup_check_screen.dart, 3 test files, README, CI workflow | Compare theme colors and sizes with Figma; run flutterfire configure and confirm "Firebase connected"; read each file so you can explain it |
