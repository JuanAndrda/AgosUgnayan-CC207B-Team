# AI_CONTEXT.md: read this first

This file tells any AI assistant (Claude, ChatGPT, Gemini, Copilot, Cursor) what this project is and the rules for helping on it. Paste it or point your assistant at it at the start of a session. The longer coding rules are in [`CLAUDE.md`](CLAUDE.md); they apply to every assistant, not only Claude.

## The project

- **AgosUgnayan**: a Flutter app (web and Android) where smallholder farmers in Leon, Iloilo sell crops directly to local buyers, with a fair PSA reference price shown before they talk.
- **Course:** CC 207B Applications Development and Emerging Technologies, WVSU BSIT, section B. A graded student project. Full requirements: [`docs/brief/CC_207B_Final_Project_Requirements.md`](docs/brief/CC_207B_Final_Project_Requirements.md). Checklist: [`docs/final-project-checklist.md`](docs/final-project-checklist.md).
- **Team:** 5 members, so the app needs **6 core features**, the AI feature included. Juan is the team leader.
- **Client and users:** smallholder farmers (main client); roles are Farmer, Buyer (household or business) and Admin (enters monthly reference prices).
- **Live app:** https://agosugnayan-cc207b.web.app (Firebase project `agosugnayan-cc207b`).

## The 6 core features

| # | Feature | Main files |
|---|---|---|
| F1 | Crop Listings (add, edit, delete with Undo, validated form) | `my_crops_screen.dart`, `crop_form_screen.dart`, `listing_service.dart` |
| F2 | Marketplace (search, filters, paging, listing details, farmer profile) | `marketplace_screen.dart`, `listing_detail_screen.dart`, `farmer_profile_screen.dart` |
| F3 | AI Listing Assistant ("Fill in for me" turns a farmer's note into form values) | `ai_assistant_box.dart`, `ai_service.dart`, `ai_listing_suggestion.dart` |
| F4 | Messages (live buyer and farmer chat) | `inbox_screen.dart`, `chat_screen.dart`, `message_service.dart` |
| F5 | Buyer Requests (buyers post needs, farmers see Buyer Demand) | `request_form_screen.dart`, `my_requests_screen.dart`, `buyer_demand_screen.dart` |
| F6 | Reference Prices (PSA farmgate prices, 12-month chart, offline copy, admin import) | `price_list_screen.dart`, `price_detail_screen.dart`, `admin_price_form_screen.dart`, `psa_service.dart` |

Supporting screens (not counted): login, register, Following, Profile, Orders, and the Buyer AI assistant.

## AI features

- All AI uses **Gemini `gemini-3.5-flash-lite`** through **Firebase AI Logic** (the `firebase_ai` package). The model name lives in one place: `AiService.modelName` in `lib/services/ai_service.dart`.
- Don't switch to `gemini-3.5-flash`: its free tier allows only 20 requests a day for the whole project.
- Features: Fill in for me (F3), Generate crop description, and the Buyer AI assistant.
- Every AI call has a loading state, a 20-second timeout, a friendly error, a manual fallback, an "AI-generated" label, and lets the user edit the result. Phone numbers and emails are removed before anything is sent (`lib/utils/privacy_filter.dart`, RA 10173).
- Details: [`docs/ai-feature-spec.md`](docs/ai-feature-spec.md) and [`docs/buyer-ai-assistant-design.md`](docs/buyer-ai-assistant-design.md).

## Hard rules

1. **Free plans only.** Firebase Spark, the Gemini free tier, and the public PSA OpenSTAT and Wikipedia APIs. Never suggest Blaze, Cloud Functions, Firebase Storage or any paid service. The workarounds we use are in [`docs/known-limitations.md`](docs/known-limitations.md).
2. **No secrets in the repo, ever.** No API keys, service accounts, `.env` files, `lib/firebase_options.dart`, `google-services.json` or `firebase.json`. Gemini has no key in the app because it goes through Firebase AI Logic. Before each commit, check: `git diff --cached | grep -iE "key|secret|token|AIza"`.
3. **Small steps.** The course AI policy forbids generating the whole app or big parts at once. Work one feature, screen or function at a time, and explain what changed, why, and how to modify it.
4. **The member must be able to explain the code.** Each member has a 2 to 3 minute code defense with a live change. Prefer simple code, and add no new package without a reason.
5. **Never write individual reflections or self or peer assessments.** Those are each member's own words.
6. **AI Usage Log.** After any AI-assisted change, give the member a line for [`docs/ai-usage-log.md`](docs/ai-usage-log.md): `| <member> | <date> | <tool> | <what was asked> | <what was generated> | <what to verify> |`
7. **Git:** feature branches (`feature/<short-name>`), small commits under the member's own account, and every PR reviewed by another member. Run `dart format .`, `flutter analyze` and `flutter test` before committing.
8. **Follow the existing UI** (`docs/prototypes/` and `lib/utils/theme.dart`). Don't redesign screens unless the team agrees.
9. **Don't push to the class org repo** (`WVSU-BSIT-Batch-2028`). Only the team leader opens the final PR there.

## Folder structure

```
lib/
  main.dart       App start, error catching, Firebase or emulator connection
  screens/        One file per page; home_screen.dart holds the tabs
  widgets/        Reusable pieces (cards, form fields, AI box, state view)
  models/         Data classes with fromMap/toMap
  services/       The only code that talks to Firebase, Gemini and web APIs
  utils/          Theme, strings, constants, validators, formatting, logger
test/             Unit and widget tests
tool/             Placeholder Firebase settings, emulator seed, rules test, AI stress test
docs/             Spec, AI Usage Log, Topic Coverage Matrix, limitations, usability review
docs/brief/       Course requirements and early planning notes (some details outdated; the code wins)
docs/prototypes/  Screen sheets of the built app
firestore.rules   Who can read and write what
```

Rule: screens never call Firebase or HTTP directly; services do.

## Open tasks (as of 2026-11-24)

- [ ] **AI sample outputs:** check the samples in `docs/ai-feature-spec.md` against the live app and add a few notes of your own.
- [ ] **Screenshots** of each core feature for the documentation (start from `docs/prototypes/`, take fresh ones if a screen changed).
- [ ] **Topic Coverage Matrix:** each feature owner checks their rows in `docs/topic-coverage-matrix.md` (file and class names still correct).
- [ ] **Known Limitations:** review `docs/known-limitations.md` before submission.
- [ ] **AI Usage Log:** every member adds their own lines.
- [ ] **Individual reflections:** each member writes their own (not AI).
- [ ] **Slides and demo:** a 5 to 10 minute deck, a live demo plus a recorded backup video, and each member ready for the code defense.

## How to run

See [`README.md`](README.md): `flutter pub get`, copy `tool/firebase_options_example.dart` to `lib/firebase_options.dart`, then use the local emulator (`--dart-define=USE_EMULATOR=true`) or the real Firebase project through `flutterfire configure`.
