# Product Implementation Plan

**Project:** Storage Unit Inventory Tracker
**Date:** February 19, 2026
**Status:** Pre-development — Planning Phase
**Framework:** React Native (Expo)
**Backend:** Supabase
**Team:** Solo developer

---

## Overview

The Storage Unit Inventory Tracker is a mobile application that allows individual consumers to catalog and remotely manage the contents of their physical storage units. The core problem it solves is uncertainty — users currently cannot know what is in their storage unit without making a physical trip. The app provides a structured, photo-rich inventory organized by unit, shelf, and container, with barcode scanning and QR code support for fast item entry and lookup.

The discovery phase is complete. The PRD defines the full feature set for v1, explicitly excluding marketplace/resale, item sharing, AI object recognition, IoT integration, and item valuation. Those are scoped to future versions. The product is B2C, targeting individual consumers who already rent storage space.

This plan covers the v1 implementation of the React Native cross-platform app backed by Supabase, with native iOS (Swift) and Android (Kotlin) apps planned for a later phase.

---

## Architecture

The app follows a **client-heavy mobile architecture** with a managed backend. There is no custom application server; all backend logic is handled via Supabase's hosted services (authentication, database, storage, and row-level security policies).

```
┌─────────────────────────────────────────────┐
│             React Native App (Expo)          │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   UI /   │  │  State   │  │ Supabase │  │
│  │  Screens │  │ (Zustand)│  │  Client  │  │
│  └──────────┘  └──────────┘  └──────────┘  │
└───────────────────────┬─────────────────────┘
                        │ HTTPS / REST + Realtime
              ┌─────────▼──────────┐
              │      Supabase       │
              │  ┌───────────────┐  │
              │  │  Auth (JWT)   │  │
              │  ├───────────────┤  │
              │  │  Postgres DB  │  │
              │  ├───────────────┤  │
              │  │  Storage      │  │
              │  │  (item photos)│  │
              │  └───────────────┘  │
              └────────────────────┘
```

**Key architectural decisions:**

- **Expo Managed Workflow** is used for the initial build. This simplifies device testing, OTA updates, and the build pipeline for a solo developer. If a native module is required that Expo doesn't support (e.g., an advanced camera library), the project can eject to a bare workflow at that point.
- **Supabase** handles auth, structured data storage (Postgres), photo/file storage, and row-level security — meaning the app can be built without writing a custom API layer.
- **Zustand** is used for client-side state management. It is lightweight, has minimal boilerplate, and is well-suited for a solo developer compared to Redux.
- **React Navigation** is used for screen routing (stack + tab navigation).
- **TypeScript** is used throughout for type safety and maintainability.
- The app is **portrait-first** and designed for phone-sized screens (no tablet layout).
- **Offline support is not in scope for v1.** The app requires a network connection to function; data is not cached for offline access.

---

## Components

The app is divided into the following logical modules. Each module corresponds to a distinct area of functionality that can be developed and tested independently.

**Authentication Module**
Handles user sign-up, login, session persistence, and password reset. Powered by Supabase Auth with email/password. The user's identity is the root anchor for all data.

**Storage Unit Manager**
Allows users to create, name, and delete storage units. Each unit can have a facility name, address, and notes. This is the top-level organizational container.

**Shelf Manager**
Within a unit, users can define shelves identified by a label (e.g., "Shelf 1", "Top Rack"). Shelves are the second tier of the location hierarchy.

**Container/Box Manager**
Boxes and containers are nested within shelves. Each container has a name and optional notes. Containers are the direct parents of items.

**Item Capture & Editor**
The primary data-entry flow. Users photograph an item, assign it a label, optionally add a description, tags, and barcode, and place it in a container. Photo upload is handled asynchronously via Supabase Storage.

**Barcode Scanner**
A camera-based scanner that reads standard product barcodes (UPC, EAN). Used to quickly populate the barcode field on an item. In v1, scanning captures the barcode number only — no product lookup is performed.

**QR Code Generator**
Generates unique QR codes for any data entity: units, shelves, boxes, or individual items. The QR code encodes a deep link back into the app. A print-friendly view is provided so users can print and physically attach the code.

**QR Code Scanner**
Scans a previously generated QR code (attached to a box, shelf, etc.) and navigates directly to that entity in the app.

**Search & Filter**
Full-text search across item names and notes. Filterable by unit, shelf, container, tag, or category. Results show the item's photo and exact location path (e.g., "Unit A → Shelf 2 → Box 3").

**Navigation Shell**
Tab and stack navigation structure that ties all modules together. Top-level tabs: My Units, Search, Scan, Profile/Settings.

**Settings & Account**
Allows users to manage their account (change email/password), view app version, and sign out.

---

## Data Model

The database lives in Supabase Postgres. Row-level security (RLS) ensures each user can only access their own data. All tables include `id` (UUID), `user_id` (foreign key to auth.users), and `created_at` timestamps.

**users**
Managed by Supabase Auth. Extended with a `profiles` table for display name and preferences if needed.

**storage_units**
```
id            UUID (PK)
user_id       UUID (FK → auth.users)
name          TEXT           -- e.g. "Unit 4B"
facility_name TEXT           -- e.g. "Public Storage Burbank"
address       TEXT
notes         TEXT
created_at    TIMESTAMPTZ
```

**shelves**
```
id            UUID (PK)
unit_id       UUID (FK → storage_units)
user_id       UUID
label         TEXT           -- e.g. "Shelf 1", "Top Rack"
position      INTEGER        -- sort order within the unit
notes         TEXT
created_at    TIMESTAMPTZ
```

**containers**
```
id            UUID (PK)
shelf_id      UUID (FK → shelves)
unit_id       UUID (FK → storage_units)
user_id       UUID
name          TEXT           -- e.g. "Box 4", "Blue Tub"
notes         TEXT
qr_code_ref   TEXT           -- unique token used in QR code URL
created_at    TIMESTAMPTZ
```

**items**
```
id            UUID (PK)
container_id  UUID (FK → containers)
shelf_id      UUID (FK → shelves)
unit_id       UUID (FK → storage_units)
user_id       UUID
name          TEXT
description   TEXT
barcode       TEXT           -- scanned product barcode, if any
photo_url     TEXT           -- Supabase Storage path
tags          TEXT[]         -- array of user-defined tags
notes         TEXT
created_at    TIMESTAMPTZ
updated_at    TIMESTAMPTZ
```

**qr_codes**
```
id            UUID (PK)
user_id       UUID
entity_type   TEXT           -- 'unit' | 'shelf' | 'container' | 'item'
entity_id     UUID           -- FK to the relevant entity
token         TEXT (UNIQUE)  -- the value encoded in the QR image
created_at    TIMESTAMPTZ
```

**Notes on the model:**
- `items` redundantly stores `shelf_id` and `unit_id` for efficient filtering without complex joins.
- The `qr_codes` table is separate to allow any entity type to receive a QR code without schema changes.
- Photos are stored in Supabase Storage under a per-user path; only the path/URL is stored in the `items` table.
- Full-text search is implemented via Postgres `tsvector` on `items.name` and `items.description`, indexed for performance.

---

## Major Technical Steps

These are the high-level implementation tasks in roughly sequential order. Each represents a meaningful, shippable slice of work.

1. **Project Bootstrap** — Initialize the Expo project with TypeScript, configure ESLint/Prettier, set up Supabase project, connect the Supabase JS client, configure environment variables.

2. **Auth Flows** — Implement sign-up, login, session persistence (via Supabase's token refresh), and password reset screens. Gate the app behind auth.

3. **Database Schema** — Create all tables in Supabase with appropriate foreign keys, indexes, and RLS policies. Enable full-text search index on items.

4. **Navigation Shell** — Set up React Navigation with bottom tabs and nested stack navigators. Define the top-level screen structure.

5. **Storage Unit CRUD** — Build the unit list screen, unit creation form, and unit detail screen. Wire to Supabase.

6. **Shelf & Container CRUD** — Build the shelf and box management screens within a unit. Maintain sort order for shelves.

7. **Item Capture Flow** — Build the item creation screen: camera capture (or photo library pick), photo upload to Supabase Storage, form for name/description/tags, and assignment to a container.

8. **Item Detail & Edit** — View full item details (photo, location path, notes) and edit any field or reassign to a different container.

9. **Barcode Scanner Integration** — Integrate a barcode scanning library (expo-barcode-scanner or Vision Camera). Capture barcode value and pre-fill the item form.

10. **QR Code Generation** — Generate a unique token per entity, store in `qr_codes` table, render a QR code image using a library. Build a print-friendly view using expo-print.

11. **QR Code Scanner & Deep Linking** — Scan a QR code and resolve the token to navigate to the correct unit, shelf, container, or item screen.

12. **Search & Filter** — Implement the search screen with real-time full-text results from Supabase. Add filter chips for unit, shelf, container, and tag.

13. **UX Polish & Error Handling** — Loading states, empty states, error toasts, permission denial handling (camera, photo library), and offline detection messaging.

14. **App Store Preparation** — App icons, splash screen, metadata, privacy policy (required by both stores for camera use), TestFlight/internal track setup, and EAS build configuration.

15. **Submission & Review** — Submit to Apple App Store and Google Play. Address any review feedback.

---

## Tools & Services

| Category | Tool / Service | Purpose |
|---|---|---|
| Framework | React Native (Expo SDK 52+) | Cross-platform mobile development |
| Language | TypeScript | Type safety throughout the codebase |
| Backend | Supabase | Auth, Postgres database, file storage |
| State Management | Zustand | Lightweight client-side state |
| Navigation | React Navigation v7 | Stack and tab navigation |
| Camera | expo-camera | Photo capture and library access |
| Barcode Scanning | expo-barcode-scanner | UPC/EAN barcode reading |
| QR Generation | react-native-qrcode-svg | Render QR code images |
| QR Printing | expo-print + expo-sharing | Generate and share print-ready QR PDFs |
| Image Handling | expo-image-picker | Photo library access |
| Image Display | expo-image | Performant cached image display |
| Build & Distribution | EAS Build (Expo Application Services) | Cloud builds for iOS and Android |
| OTA Updates | EAS Update | Push JS-layer updates without app store review |
| Version Control | GitHub | Source control |
| Linting | ESLint + Prettier | Code quality and formatting |
| Secrets | EAS Secrets / .env | Supabase keys and environment config |

---

## Risks & Unknowns

| Risk | Severity | Notes |
|---|---|---|
| Camera permission handling differences (iOS vs Android) | Medium | Android and iOS have different permission flows and camera APIs. Must test on both platforms early to avoid late-stage surprises. |
| Expo Managed Workflow limitations | Medium | Some advanced camera features (e.g., continuous scan, custom overlays) may require ejecting to bare workflow or using Expo Dev Client. Evaluate early. |
| Supabase Storage cost at scale | Low–Medium | Each item photo could be 1–5MB. At scale, storage costs grow. Consider client-side image compression before upload to control this. |
| QR code printing reliability | Medium | Printing from a mobile app depends on AirPrint (iOS) or system print dialog (Android). The experience varies by device. May need to fall back to a "share as PDF" approach. |
| Full-text search performance | Low | Supabase Postgres supports indexed full-text search, but the query must be tuned for the data shape. Performance should be tested with realistic data volumes. |
| App Store review time | Medium | Apple App Store reviews can take 1–3+ days, with potential rejection requiring re-submission. Camera usage requires a clearly stated privacy policy and usage description strings. |
| Barcode library coverage | Low | expo-barcode-scanner covers standard formats (UPC-A, EAN-13, Code 128, QR). Edge cases (damaged barcodes, unusual formats) may not scan reliably. |
| Offline behavior is undefined | Low | The discovery doc does not mention offline mode. The plan assumes connectivity is required. If users expect offline access, this is a significant future scope item. |
| Photo storage path security | Low | Supabase Storage bucket policies must be set so users can only access their own photos. This needs to be verified during schema and storage setup. |
| Solo developer bus factor | Medium | All knowledge and implementation is concentrated in one person. Documentation and commit hygiene are important mitigations. |

---

## Milestones

Development is phased to deliver working, testable increments. Durations are estimates for a solo developer working full-time.

**Phase 1 — Foundation (Weeks 1–3)**
Project is bootstrapped, auth works end-to-end, and basic CRUD for storage units, shelves, and containers is functional. Data persists to Supabase and is scoped per user via RLS.

Deliverables: working app with auth, unit/shelf/container management, Supabase schema live.

**Phase 2 — Item Capture (Weeks 4–6)**
Users can photograph items, label them, assign them to a container, and view them. Photos upload to Supabase Storage. Item editing and deletion work.

Deliverables: full item capture flow, photo storage, item detail screen.

**Phase 3 — Scanning (Weeks 7–8)**
Barcode scanning is integrated into the item creation flow. QR codes can be generated for any entity and rendered as a printable/shareable output. QR scanning navigates to the target entity.

Deliverables: barcode scanner, QR generation + print view, QR scanner with deep linking.

**Phase 4 — Search & Discovery (Weeks 9–10)**
Full-text search is live. Filter chips allow narrowing by unit, shelf, container, and tag. Search results show item photo and full location path.

Deliverables: search screen, filter UI, indexed search queries in Supabase.

**Phase 5 — Polish & Launch Prep (Weeks 11–12)**
UX gaps are addressed: empty states, loading indicators, error handling, offline messaging, permission denial flows. App icons, splash screen, and store metadata are finalized. EAS builds are configured for both platforms.

Deliverables: production-ready builds for iOS and Android, App Store and Play Store submissions.

---

## Target Device Sizes

The app is designed for **phones in portrait orientation only**. Tablet and landscape layouts are not supported in v1.

**iOS (iPhone)**
| Size Class | Example Devices | Logical Width |
|---|---|---|
| Small | iPhone SE (3rd gen) | 375pt |
| Standard | iPhone 15, 15 Pro | 390–393pt |
| Large | iPhone 15 Plus, 15 Pro Max | 430pt |

- Minimum iOS version: **iOS 16.0**
- All layouts must be tested on the small (375pt) and large (430pt) ends of the range.
- Safe area insets (notch, Dynamic Island, home indicator) must be respected using `SafeAreaProvider`.

**Android**
| Size Class | Example Devices | dp Width |
|---|---|---|
| Compact | Pixel 4a, mid-range Android | 360dp |
| Standard | Pixel 8, Samsung Galaxy S24 | 384–411dp |
| Large | Samsung Galaxy S24+, Pixel 9 Pro XL | 412dp |

- Minimum Android SDK: **API 26 (Android 8.0)**
- The app must handle varying pixel densities (mdpi to xxxhdpi).
- System bar insets (status bar, navigation bar) must be accounted for on both gesture-nav and button-nav Android devices.

---

## Environment Setup

Prerequisites and steps to get a local development environment running.

**Required Tools**
- Node.js 20 LTS or later
- npm or yarn
- Expo CLI: `npm install -g expo-cli`
- EAS CLI: `npm install -g eas-cli`
- Xcode (macOS only, for iOS Simulator)
- Android Studio (for Android emulator) with an AVD configured
- A Supabase account and project

**Steps**

1. Clone the repository and install dependencies:
   ```bash
   git clone <repo-url>
   cd storage-tracker
   npm install
   ```

2. Create a `.env` file in the project root with the Supabase credentials:
   ```
   EXPO_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
   EXPO_PUBLIC_SUPABASE_ANON_KEY=<your-anon-key>
   ```

3. In Supabase, run the schema migration scripts (located in `/supabase/migrations`) to create all tables, indexes, and RLS policies.

4. Configure Supabase Storage with a `photos` bucket. Set the bucket to private (authenticated users only) and apply the per-user access policy.

5. Start the Expo development server:
   ```bash
   npx expo start
   ```
   Use the Expo Go app on a physical device, or press `i` for iOS Simulator / `a` for Android emulator.

6. For a production build (required for native modules and app store submission):
   ```bash
   eas build --platform ios
   eas build --platform android
   ```

**Notes**
- Camera and barcode scanning do not function in the iOS Simulator. A physical device is required for testing those features.
- Supabase RLS policies must be enabled before testing multi-account scenarios; local testing with a single user will work without them but will not reflect production security behavior.

---

## Future Plans

The following items are explicitly out of scope for v1 but are intended to follow once the cross-platform app is stable and validated.

**Native iOS App (Swift)**
A fully native iOS application built in Swift/SwiftUI using the same Supabase backend and data model defined in this plan. The native app would target performance improvements, deeper iOS integration (Shortcuts, Spotlight search, Widgets), and a fully platform-native UX.

**Native Android App (Kotlin)**
A fully native Android application built in Kotlin using Jetpack Compose. Would share the same backend and data model. Enables deeper Android integration (home screen widgets, intent sharing, scoped storage).

**AI Item Recognition**
Use the item photos already stored in Supabase Storage as input to a vision model to automatically suggest item names, categories, and tags — reducing manual entry.

**Item Valuation & Market Lookup**
Surface estimated resale or market values for catalogued items by querying product databases or resale marketplaces. Identified in the discovery doc as a future monetization and engagement lever.

**Sharing & Collaboration**
Allow a storage unit to be shared with another user (e.g., a family member or co-renter), with read or read/write access controls.

**Offline Mode**
Cache inventory data locally so users can browse their inventory without a network connection. Changes made offline would sync when connectivity is restored.
