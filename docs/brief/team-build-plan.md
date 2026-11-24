# AgosUgnayan: team build plan (who builds what, and how we PR)

Member 1 is Juan, the team leader. Members 2 to 5 are placeholders until we have names and GitHub usernames.

## Why the code isn't all written at once
The course AI policy (and our own CLAUDE.md) forbids generating the whole app in one go. Each member also has to explain their own code and make a small live change in a 2 to 3 minute defense. So I write the code **one feature at a time, on that member's branch, with that member**, and explain each file as it lands. This plan makes sure everyone knows their part before we start.

## 1. Who owns what

| Member | Owns | Why this split |
|---|---|---|
| **1 (Juan, lead)** | Setup (project, theme, routing, shared widgets, CI) and **F6 Reference Prices** + admin price form | The lead sets the base everyone builds on. Prices are a small read-mostly feature, which leaves room for lead tasks. |
| **2** | Register, login, role homes, settings, and **F5 Buyer Requests** | Auth plus a form-and-list feature, similar in style to F1. |
| **3** | **F1 Crop Listings** and **F4 Messages** | F1 is the main validated form. Messages reuses the same service pattern. |
| **4** | **F2 Marketplace** and the final **Docs** PR | Lists, search, filters and pagination. Docs needs someone who has seen every screen as a buyer. |
| **5** | **F3 AI Listing Assistant** | The AI feature needs the most explaining (prompt, validation, fallback, privacy), so it gets one dedicated owner. |

Everyone owns at least one counted core feature to defend.

## 2. PR order and reviewers

Each PR waits until the one before it is merged. The reviewer is a different member, who must approve it before merge.

| PR | Branch | Owner | Reviewer | Needs merged first |
|---|---|---|---|---|
| 1 | `feature/setup` | M1 | M2 | none |
| 2 | `feature/auth` | M2 | M3 | PR 1 |
| 3 | `feature/crop-listings` (F1) | M3 | M4 | PR 2 |
| 4 | `feature/marketplace` (F2) | M4 | M5 | PR 3 |
| 5 | `feature/ai-assistant` (F3) | M5 | M1 | PR 3 (adds to the Add Crop form) |
| 6 | `feature/messages` (F4) | M3 | M2 | PR 4 |
| 7 | `feature/buyer-requests` (F5) | M2 | M4 | PR 6 (the "Message buyer" button) |
| 8 | `feature/reference-prices` (F6) | M1 | M5 | PR 3 (price hint on Add Crop) |
| 9 | `feature/docs` | M4 | M1 | all |

Commit rules for every PR:
- Commit on your own laptop, under your own GitHub account, a few small commits per PR (for example `Add quantity validator`, `Show Undo after delete`). Don't push one big commit.
- Before every commit: `dart format .`, `flutter analyze` (0 issues), `flutter test` (all pass), and check the diff has no keys.
- In the PR description, include what changed, how to test it, and your AI Usage Log line.
- The reviewer runs the app, tries the feature, and leaves at least one real comment before approving.

## 3. Each member's files and what to learn

Keep every file short (under about 150 lines) and commented. Screens call services, and services call Firebase.

### M1: Setup (PR 1)
- Files: `main.dart`, `utils/theme.dart` (taken from Figma v3), `utils/strings.dart`, `utils/validators.dart`, `utils/app_logger.dart`, `widgets/async_state_view.dart`, `widgets/confirm_dialog.dart`, `app_router.dart` (go_router), `.github/workflows/ci.yml`, README.
- Learn: how `ThemeData` works, how go_router routes and redirects, the 4 states of `AsyncStateView`, and what CI does.
- Practice live change: change the primary color, or add a new route.

### M1: F6 Reference Prices (PR 8)
- Files: `models/price_record.dart`, `services/price_service.dart` (Firestore plus a shared_preferences cache), `screens/price_list_screen.dart`, `screens/price_detail_screen.dart` (fl_chart), `screens/admin_price_form_screen.dart`.
- Learn: reading from Firestore, saving JSON in shared_preferences for offline use, and drawing a `LineChart`.
- Practice: show 6 months instead of 12, or add a field to the admin form.

### M2: Auth and homes (PR 2)
- Files: `models/app_user.dart`, `services/auth_service.dart`, `screens/login_screen.dart`, `screens/register_screen.dart`, `screens/farmer_home_screen.dart`, `screens/buyer_home_screen.dart`, `screens/admin_home_screen.dart`, `screens/settings_screen.dart`.
- Learn: Firebase Auth sign-in and sign-up, saving the user profile in `users/{uid}`, sending each role to its own home, and the logout confirmation.
- Practice: add a field to Register, or change where a role lands after login.

### M2: F5 Buyer Requests (PR 7)
- Files: `models/buyer_request.dart`, `services/request_service.dart`, `screens/request_form_screen.dart`, `screens/my_requests_screen.dart`, `screens/buyer_demand_screen.dart`, `screens/request_detail_screen.dart`.
- Learn: a second validated form, how "Closed" is calculated from the needed-by date, and filtering by status.
- Practice: add a "Notes" length limit, or change the Closed rule.

### M3: F1 Crop Listings (PR 3)
- Files: `models/crop.dart` (the fixed crop and unit list), `models/listing.dart`, `services/listing_service.dart`, `screens/my_crops_screen.dart`, `screens/crop_form_screen.dart` (add and edit), `widgets/listing_card.dart`, `test/validators_test.dart`, `test/crop_form_test.dart`.
- Learn: `Form` + `GlobalKey<FormState>`, validators, `PopScope` for the discard warning, delete with an Undo SnackBar, and `mounted` checks.
- Practice: add a new form field, or change a validation message.

### M3: F4 Messages (PR 6)
- Files: `models/chat.dart`, `models/message.dart`, `services/message_service.dart`, `screens/inbox_screen.dart`, `screens/chat_screen.dart`.
- Learn: Firestore `snapshots()` streams, `StreamBuilder`, why the chat id is the two user ids sorted, and loading older messages on scroll.
- Practice: show the time under each message, or change the page size.

### M4: F2 Marketplace (PR 4)
- Files: `screens/marketplace_screen.dart`, `screens/listing_detail_screen.dart`, `screens/farmer_profile_screen.dart`, `widgets/filter_bar.dart`, plus a `fetchPage()` method in `listing_service.dart`.
- Learn: pagination with `.limit(20)` and `.startAfterDocument`, a debounced search, `RefreshIndicator`, and passing an id through a route.
- Practice: change the page size, or add a new filter chip.

### M4: Docs (PR 9)
- Files: README, `docs/ai-feature-spec.md` (written together with M5), `docs/known-limitations.md`, `docs/planned-features.md`, `docs/screenshots.md`, `docs/demo-checklist.md`, and the final pass on `docs/topic-coverage-matrix.md`.

### M5: F3 AI Listing Assistant (PR 5)
- Files: `services/ai_service.dart` (firebase_ai, 20 s timeout, JSON parsing), `utils/privacy_filter.dart` (removes phone numbers and emails), `models/ai_listing_suggestion.dart` (validates each field), `widgets/ai_assistant_box.dart`, `widgets/ai_suggestion_chip.dart`, `test/privacy_filter_test.dart`, `test/ai_listing_suggestion_test.dart`.
- Learn: what the prompt says and why, why invalid fields are dropped, the fallback path, and why AI beats fixed rules here.
- Practice: add a new unit word (for example "bugkos"), or change the timeout.

## 4. Shared docs everyone updates
- `docs/ai-usage-log.md`: one line per AI-assisted change, in the CLAUDE.md format, written by the member who asked.
- `docs/topic-coverage-matrix.md`: the feature owner adds their rows in their own PR.
- Individual reflections and peer assessments are each member's own words. I won't write them.

## 5. Still needed before PR 1
1. The repo: the fork URL, or "start fresh" in `C:\dev\agosugnayan-cc207b`.
2. Real names and GitHub usernames for Members 2 to 5, or a change to the split above.
3. Figma frames for Register, Settings and the bottom navs before PR 2. The other missing frames are needed only before their own feature.
