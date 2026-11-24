# AgosUgnayan (CC 207B) — Step 0 plan

Status: plan only, nothing built yet. Waiting for your OK and answers in section 6.

## 1. What I understood (under 20 lines)

1. AgosUgnayan connects smallholder farmers in Iloilo (starting with Leon) directly with local buyers, and shows a fair reference price before they talk.
2. Three roles: Farmer (main user and client), Buyer (one role with a Household/Business label), Admin (client staff who only enters monthly prices).
3. No payment, cart, delivery, ratings, map, follow, alerts, reports, photos or other languages. Those go under "Planned features".
4. Every price shows source and month ("PSA farmgate, Iloilo Province, Aug 2026"), never "live". Asking and target prices look different from reference prices.
5. Six core features, built in this order, one PR each: Crop Listings, Marketplace, AI Listing Assistant, Messages, Buyer Requests, Reference Prices.
6. The AI feature turns a farmer's mixed-language note into suggested form fields through Gemini via `firebase_ai` + App Check. It only extracts, never invents a price. 20 s timeout, fallback to the manual form, phone numbers and emails removed before sending.
7. Stack: Flutter stable (web + Android), Firebase Auth (email/password), Firestore, Firebase AI Logic, App Check, and only the 10 listed packages.
8. Screens never call Firebase; services do. All text in `strings.dart`, all colors and sizes in `theme.dart`, one shared `AsyncStateView`.
9. Design comes from the v3 row of the Figma file. Where a needed screen has no frame, or code and Figma disagree, I stop and ask.
10. Work is graded from GitHub: one feature at a time, an owner per feature on `feature/<short-name>`, a plan before and a summary after each feature, format/analyze/test before every commit, an AI Usage Log line after every change, Topic Coverage Matrix after every feature.
11. I never write reflections or peer assessments.

## 2. Firestore data model (proposed)

| Collection | Key fields | Notes |
|---|---|---|
| `users/{uid}` | `role` (farmer, buyer, admin), `displayName`, `town`, `buyerType` (household, business; buyers only), `consentAt`, `createdAt` | No phone number stored. `admin` can't be chosen at sign-up; it is set by hand in the Firebase console and rules block role changes. |
| `listings/{id}` | `farmerId`, `farmerName`, `cropId`, `quantity`, `unit`, `askingPrice`, `availableDate`, `town`, `description`, `status` (active, sold), `aiAssisted`, `createdAt`, `updatedAt` | Delete is a real delete; Undo writes the same document back. |
| `requests/{id}` | `buyerId`, `buyerName`, `buyerType`, `cropId`, `quantity`, `unit`, `targetPrice` (or min/max, see 6.4), `neededBy`, `town`, `notes`, `createdAt` | "Closed" is worked out from `neededBy` when read, so no server code is needed. |
| `chats/{uidA_uidB}` | `participants` [uidA, uidB], `names` map, `lastMessage`, `lastMessageAt`, `lastSenderId`, `topic` (listing or request id + crop) | The id is the two user ids sorted, so each pair gets exactly one chat. |
| `chats/{id}/messages/{id}` | `senderId`, `text`, `createdAt` | Loaded newest first, 20 at a time on scroll up. |
| `prices/{cropId}/months/{yyyy-MM}` | `price`, `unit`, `source` ("PSA OpenSTAT farmgate"), `area` ("Iloilo Province"), `month`, `enteredBy`, `enteredAt` | Only admin can write. |
| `prices/{cropId}` | `latestPrice`, `latestMonth`, `previousPrice` | Summary row for the price list, updated by the admin form in the same batch. |

Crops are a fixed list in code (`lib/models/crop.dart`) with local names (talong, kamatis, repolyo…) and allowed units (kg, sako, kaban, tali). The AI validator, the dropdowns and the price list all use it. Composite indexes go in `firestore.indexes.json`.

## 3. Feature owners (names needed)

| # | Work | Branch | Owner | Reviewer |
|---|---|---|---|---|
| 1 | Setup: project, Firebase link, theme, routing, AsyncStateView, validators, strings, README, CI | `feature/setup` | ? | ? |
| 2 | Register, login, role homes, settings | `feature/auth` | ? | ? |
| 3 | F1 Crop Listings | `feature/crop-listings` | ? | ? |
| 4 | F2 Marketplace | `feature/marketplace` | ? | ? |
| 5 | F3 AI Listing Assistant | `feature/ai-assistant` | ? | ? |
| 6 | F4 Messages | `feature/messages` | ? | ? |
| 7 | F5 Buyer Requests | `feature/buyer-requests` | ? | ? |
| 8 | F6 Reference Prices | `feature/reference-prices` | ? | ? |
| 9 | Docs (AI Feature Spec, limitations, screenshots, demo checklist) | `feature/docs` | ? | ? |

Suggestion: give each of the 5 members one of F1 to F6 (one person takes two), and spread setup, auth and docs so everyone has commits early. F3 builds on the F1 form, so F1 must merge first.

## 4. Screens and their v3 Figma frames

Marked **NO FRAME** where nothing exists yet, and **CHANGE** where the frame shows things we are not building or that break a rule.

### Supporting (not counted)
| Screen | v3 frame | Gap |
|---|---|---|
| Login | 01 Login · v3 | CHANGE: field says "Phone number or email" but we use email only. Has an "I am a Farmer/Buyer" toggle; the role should come from the account. |
| Register (privacy notice, consent, Household/Business) | **NO FRAME** | Only a "Register" tab exists on 01. |
| Farmer home | 02 Seller Dashboard · v3 | CHANGE: remove "Recent alerts" and the Alerts tab; price footer "Updated Sep 10" must become "PSA farmgate, Iloilo Province, Aug 2026". |
| Buyer home | 10 Buyer Dashboard · v3 | CHANGE: remove "Farmers you follow"; "Save ₱x/kg vs DA retail" needs DA retail prices we don't collect. |
| Settings with logout | **NO FRAME** | 18 Language Settings is only a language picker (not building). |
| Admin price list + price form | **NO FRAME** | |

### Core features
| Feature | Screen | v3 frame | Gap |
|---|---|---|---|
| F1 | My Crops | 06 My Crops · v3 | CHANGE: tabs Active / Sold out / Removed and a "Remove" button; we need "Mark as sold" and "Delete" (with Undo), so "Removed" goes away. |
| F1 | Add / Edit Crop | 07 Add Crop · v3 | CHANGE: remove photos; no Edit title variant; label is "Harvest date" (prompt says available date). |
| F2 | Marketplace | 11 Marketplace · v3 | CHANGE: drop List/Map toggle; keep crop + town filters ("Any price" and "Sort" not in scope). |
| F2 | Listing details | 12 Crop Listing Details · v3 | CHANGE: drop "Make an offer"; "Retail ref. (DA)" becomes the PSA reference price with source and month. |
| F2 | Farmer profile | 13 Farmer Profile · v3 | CHANGE: drop "Follow". |
| F3 | AI assistant box on Add Crop, with "AI suggestion" chips, privacy line, loading and error states | **NO FRAME** | |
| F4 | Inbox | 16 Messages-Chat · v3 | Fine (inbox only). |
| F4 | Conversation | **NO FRAME** | |
| F5 | Post / Edit Request | 14 Post Buyer Request · v3 | CHANGE: Figma has target price min and max; prompt says one target price. |
| F5 | My Requests | 15 My Requests · v3 | CHANGE: "Answered" status and "farmers sent offers" have no data behind them; no edit/delete actions shown. |
| F5 | Buyer Demand (farmer) | 05 Buyer Demand · v3 | Fine. |
| F5 | Request details | 08 Buyer Profile-Request Details · v3 | Small: "Message" and "Contact buyer" do the same thing. |
| F6 | Reference price list | 03 Market Prices · v3 | CHANGE: rows show towns (Leon, Pototan…) and daily "Updated Sep 10"; PSA data is province-level and monthly. Needs a "Saved copy from <date>" offline banner. |
| F6 | Crop price details (12-month chart) | 04 Crop Details · v3 | CHANGE: drop 3M/6M/1Y toggle, DA retail price and "Set price alert". |
| F6 | Reference price under the Add Crop price field | **NO FRAME** | Part of the 07 redesign. |

### Bottom navigation
- Farmer (v3): Home, Prices, Demand, My crops, Alerts. Alerts is not built and there is no Messages tab.
- Buyer (v3): Home, Market, Requests, Account. No Prices and no Messages tab.
- Both need a change in Figma before step 2.

### Frames we ignore (Planned features)
09 Farmer Alerts, 17 Followed Sellers, 19 Map View, 18 Language Settings.

### Also missing for the docs
- The file has v1, v2 and v3 hi-fi rows, but I did not find low-fidelity wireframes. The CC 207B docs need both for every screen.
- The Notes page says no web layout exists. My default: on web, show the mobile layout centered at phone width.

## 5. Risks

1. **Firebase config in the repo.** `firebase_options.dart` and `google-services.json` hold Firebase API keys. They are not true secrets, but your rule is "no API key anywhere in the repo". My default: gitignore them, commit a placeholder `firebase_options.example.dart` so CI can analyze and test, and have each member run `flutterfire configure` (README steps). The FlutterFire CLI isn't installed on this laptop yet.
2. **App Check.** Web needs a reCAPTCHA Enterprise key (passed with `--dart-define`, not in code). Android in development uses App Check debug tokens registered in the console.
3. **Search.** Firestore has no text search. Marketplace search filters by crop and town in the query and matches text only within loaded pages. This goes in Known limitations.
4. **Indexes.** Paginated, filtered lists need composite indexes. I'll keep them in `firestore.indexes.json` so anyone can deploy them.
5. **Admin.** The admin account is promoted by hand in the console; the docs will say how.
6. **AI quality.** Hiligaynon units and local crop names may be misread. The validator drops anything not in the crop list, and the farmer checks every field.

## 6. What I need from you

1. **Repo.** Is there a final-project repo in the WVSU-BSIT-Batch-2028 org you forked? If yes, send the fork URL and I'll clone it into `C:\dev\agosugnayan-cc207b`. If not, I'll create the folder with `git init` and you add the remote later.
2. **Owners.** The 5 members' names and GitHub usernames, and who owns each row in section 3.
3. **Figma.** Add the NO FRAME screens and fix the CHANGE frames before their feature starts. Register, Settings and both bottom navs are needed before step 2. Or tell me which CHANGE items should follow Figma instead of the prompt.
4. **Target price.** One value (prompt) or min and max (Figma)? I recommend min and max: it matches Figma and the buyer cards.
5. **Firebase config.** OK to gitignore `firebase_options.dart` (risk 1)? I recommend yes.
