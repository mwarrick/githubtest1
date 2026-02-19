# Feature Backlog — Storage Unit Inventory Tracker

**Project:** Storage Unit Inventory Tracker
**Generated from:** plan.md
**Date:** February 19, 2026
**Framework:** React Native (Expo) + Supabase

---

## Phase 1 — Foundation (Weeks 1–3)

**Goal:** Working app with auth, unit/shelf/container management, Supabase schema live.

### Project Bootstrap

- [ ] Initialize Expo project with TypeScript template
- [ ] Configure ESLint and Prettier with project rules
- [ ] Set up Supabase project (create org, project, and region)
- [ ] Install and configure Supabase JS client (`@supabase/supabase-js`)
- [ ] Set up `.env` file with `EXPO_PUBLIC_SUPABASE_URL` and `EXPO_PUBLIC_SUPABASE_ANON_KEY`
- [ ] Configure EAS CLI and link to Expo account
- [ ] Set up GitHub repository with initial commit and `.gitignore`
- [ ] Verify dev server runs on iOS Simulator and Android emulator

### Auth Flows

- [ ] Build Sign Up screen (email + password)
- [ ] Build Login screen (email + password)
- [ ] Implement session persistence using Supabase token refresh
- [ ] Build Forgot Password / Password Reset screen
- [ ] Gate all app routes behind authenticated session
- [ ] Handle auth errors (invalid credentials, unconfirmed email, etc.)
- [ ] Implement Sign Out action from Settings

### Database Schema

- [ ] Create `storage_units` table with RLS policy (user-scoped)
- [ ] Create `shelves` table with RLS policy
- [ ] Create `containers` table with RLS policy
- [ ] Create `items` table with RLS policy
- [ ] Create `qr_codes` table with RLS policy
- [ ] Add indexes on `user_id` foreign keys across all tables
- [ ] Enable and configure full-text search (`tsvector`) index on `items.name` and `items.description`
- [ ] Configure Supabase Storage `photos` bucket (private, per-user access policy)
- [ ] Write and version migration scripts in `/supabase/migrations`

### Navigation Shell

- [ ] Install and configure React Navigation v7
- [ ] Set up bottom tab navigator with tabs: My Units, Search, Scan, Profile/Settings
- [ ] Set up stack navigators nested within each tab
- [ ] Define top-level screen registry and route names in TypeScript
- [ ] Configure deep link URL scheme for QR code navigation

### Storage Unit CRUD

- [ ] Build Unit List screen (shows all user units)
- [ ] Build Unit Creation form (name, facility name, address, notes)
- [ ] Build Unit Detail screen (shows shelves within unit)
- [ ] Implement Unit Edit (update fields)
- [ ] Implement Unit Delete (with confirmation prompt)
- [ ] Wire all unit operations to Supabase

### Shelf & Container CRUD

- [ ] Build Shelf list view within Unit Detail screen
- [ ] Build Shelf creation form (label, position/sort order, notes)
- [ ] Implement Shelf Edit and Delete
- [ ] Build Container list view within Shelf Detail screen
- [ ] Build Container creation form (name, notes)
- [ ] Implement Container Edit and Delete
- [ ] Store and maintain `position` sort order for shelves within a unit
- [ ] Wire all shelf and container operations to Supabase

---

## Phase 2 — Item Capture (Weeks 4–6)

**Goal:** Full item capture flow, photo storage, item detail screen.

### Item Capture Flow

- [ ] Build Item Creation screen with container assignment selector
- [ ] Integrate `expo-camera` for in-app photo capture
- [ ] Integrate `expo-image-picker` for photo library selection
- [ ] Implement async photo upload to Supabase Storage (per-user path)
- [ ] Store `photo_url` (Supabase Storage path) on item record
- [ ] Build form fields: name, description, tags (multi-value), notes
- [ ] Implement client-side image compression before upload to manage storage costs
- [ ] Handle camera and photo library permission prompts (iOS + Android)
- [ ] Show upload progress indicator during photo upload

### Item Detail & Edit

- [ ] Build Item Detail screen: photo, name, description, tags, notes, location path
- [ ] Display full location path (Unit → Shelf → Container) on detail screen
- [ ] Build Item Edit screen (all fields editable)
- [ ] Allow reassignment of item to a different container
- [ ] Implement Item Delete (with confirmation prompt)
- [ ] Use `expo-image` for performant cached image display

---

## Phase 3 — Scanning (Weeks 7–8)

**Goal:** Barcode scanner, QR generation + print view, QR scanner with deep linking.

### Barcode Scanner Integration

- [ ] Integrate `expo-barcode-scanner` (or evaluate Vision Camera as alternative)
- [ ] Build Barcode Scanner screen with camera overlay UI
- [ ] Support UPC-A, EAN-13, and Code 128 barcode formats
- [ ] On successful scan, pre-fill barcode field in Item Creation form
- [ ] Handle permission denial for camera (barcode scanner path)
- [ ] Handle scan failures and unrecognized barcode formats gracefully

### QR Code Generation

- [ ] Generate unique token per entity (unit, shelf, container, item) on creation
- [ ] Store token in `qr_codes` table linked to entity type and ID
- [ ] Integrate `react-native-qrcode-svg` to render QR code image
- [ ] Build QR Code view screen (shows QR image for any entity)
- [ ] Build print-friendly view using `expo-print`
- [ ] Implement "Share as PDF" fallback using `expo-sharing` for devices without AirPrint/system print
- [ ] Allow QR code generation to be triggered from Unit, Shelf, Container, and Item detail screens

### QR Code Scanner & Deep Linking

- [ ] Build QR Scanner screen (reuses camera, distinct from barcode scanner)
- [ ] Resolve scanned token against `qr_codes` table to identify entity type and ID
- [ ] Navigate to the correct detail screen based on resolved entity
- [ ] Handle invalid or unrecognized QR tokens (error state)
- [ ] Configure deep link URL scheme to support navigation from external QR scan (system camera)

---

## Phase 4 — Search & Discovery (Weeks 9–10)

**Goal:** Search screen, filter UI, indexed search queries in Supabase.

### Search & Filter

- [ ] Build Search screen with real-time text input
- [ ] Implement full-text search query against `items` via Supabase (using `tsvector` index)
- [ ] Display search results with item photo thumbnail and full location path (Unit → Shelf → Container)
- [ ] Add filter chips: by Unit, by Shelf, by Container, by Tag
- [ ] Support combining multiple active filters simultaneously
- [ ] Show empty state when no results match
- [ ] Debounce search input to avoid excessive Supabase queries
- [ ] Test search performance with realistic data volumes (50–500 items)

---

## Phase 5 — Polish & Launch Prep (Weeks 11–12)

**Goal:** Production-ready builds for iOS and Android, App Store and Play Store submissions.

### UX Polish & Error Handling

- [ ] Add loading indicators for all async operations (fetches, uploads, saves)
- [ ] Design and implement empty states for: unit list, shelf list, container list, item list, search results
- [ ] Implement error toast/snackbar component for failed network operations
- [ ] Add offline detection and display a user-facing message when no connection is available
- [ ] Handle camera permission denial gracefully (prompt user to open Settings)
- [ ] Handle photo library permission denial gracefully
- [ ] Validate all forms with user-friendly inline error messages
- [ ] Ensure safe area insets are applied correctly across all screens (notch, Dynamic Island, home indicator)
- [ ] Test layout on small (375pt/360dp) and large (430pt/412dp) screen sizes
- [ ] Verify system bar insets on gesture-nav and button-nav Android devices

### App Store Preparation

- [ ] Design and export app icon assets (all required sizes for iOS and Android)
- [ ] Configure splash screen
- [ ] Write App Store description, keywords, and screenshots (iOS)
- [ ] Write Play Store description, screenshots, and feature graphic (Android)
- [ ] Write Privacy Policy (required for camera usage on both stores)
- [ ] Add iOS usage description strings (`NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`)
- [ ] Configure `app.json` / `eas.json` for production builds
- [ ] Set up EAS Build for iOS (requires Apple Developer account + provisioning profile)
- [ ] Set up EAS Build for Android (requires keystore)
- [ ] Configure EAS Update for OTA JS-layer updates post-launch
- [ ] Set minimum iOS version to iOS 16.0 in build config
- [ ] Set minimum Android SDK to API 26 in build config
- [ ] Run production builds on physical devices for final QA

### Submission & Review

- [ ] Submit iOS build to TestFlight for internal testing
- [ ] Submit Android build to internal test track on Google Play Console
- [ ] Address any build rejection or review feedback from Apple
- [ ] Address any policy issues raised by Google Play review
- [ ] Submit final builds for public release on App Store and Google Play

---

## Backlog Summary

| Phase | Tasks |
|---|---|
| Phase 1 — Foundation | 36 tasks |
| Phase 2 — Item Capture | 15 tasks |
| Phase 3 — Scanning | 18 tasks |
| Phase 4 — Search & Discovery | 8 tasks |
| Phase 5 — Polish & Launch Prep | 25 tasks |
| **Total** | **102 tasks** |
