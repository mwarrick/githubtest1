# Discovery Document

**Project:** Storage Unit Inventory Tracker
**Date:** February 19, 2026
**Status:** Pre-development — Discovery Phase

---

## Summary

A mobile application that enables consumers to catalog and manage the contents of their personal storage units. The app solves a common pain point for people who rent storage space: not knowing what they have stored without making a physical trip to the unit. By providing a structured, photo-rich inventory system organized by unit, shelf, and container, users can remotely look up items, locate them precisely, and manage multiple storage spaces from a single app.

---

## Goals

The primary goal is to allow users to know exactly what is in their storage unit at any time — without visiting it. Secondary goals include:

- Eliminating wasted trips caused by uncertainty about what is stored
- Reducing duplicate purchases by surfacing what users already own
- Supporting users who manage multiple storage units in one unified experience
- Laying the foundation for future item valuation based on photo recognition

---

## Target Users

The app is designed for **individual consumers (B2C)** who currently rent one or more physical storage units. This is not a product aimed at businesses or warehouse operators.

Key user characteristics:

- Already rents at least one storage unit
- Willing to invest time upfront to catalog their items
- Expects their data to persist and sync across devices (cloud sync assumed)
- May manage multiple units at the same location or different facilities
- Is not expected to be highly technical, but is comfortable using a smartphone camera

---

## Key Features

### Multiple Storage Units
Users can create and manage several storage units within the app. Each unit is a top-level organizational container.

### Shelf & Location Tracking
Within each unit, items can be assigned to specific shelves identified by number or label, enabling precise location information (e.g., "Unit B → Shelf 3 → Box 4").

### Box / Container Grouping
Items are organized inside named boxes or containers, which are themselves placed on shelves within a unit.

### Photo-Based Item Capture
Users photograph items to create visual inventory records. Each item entry includes a photo, a name/label, and optional notes.

### Search & Filter
Full-text search across all items, filterable by unit, shelf, container, category, or tag — so users can quickly find any item by name or attribute.

### Barcode & QR Code Scanning
Users can scan standard barcodes (product barcodes) to log items quickly. The system will also generate unique QR codes that can be printed and physically attached to any data element (a unit, shelf, box, or individual item) for fast lookup.

### Cloud Sync
All inventory data is synced to the cloud, ensuring persistence across devices and protection against data loss.

---

## Success Criteria

The primary success metric for v1 is **active item logging** — users are genuinely cataloguing their storage and returning to reference or update it. Supporting indicators include:

- A meaningful percentage of registered users create at least one full inventory (unit + shelf + items)
- Users return to the app organically between storage visits
- Core flows (add item, search item, scan QR) can be completed without confusion or support requests
- Positive app store ratings reflecting that the core problem feels solved

---

## Out of Scope (v1)

The following are explicitly excluded from the first version to maintain focus and reduce scope creep:

| Feature | Rationale |
|---|---|
| Marketplace / resale | Users cannot list, sell, or price items from within the app |
| Sharing & collaboration | Inventory is private; no multi-user or shared unit access |
| Smart home / IoT integration | No connected locks, cameras, or facility integrations |
| AI object recognition | Photos are user-labeled; no automatic item identification in v1 |
| Item valuation / market lookups | Planned for a future version once the photo inventory foundation is in place |

---

## User Stories

The following high-level stories capture the core user needs that drive the feature set.

**Inventory Setup**
As a storage unit renter, I want to set up my unit in the app with shelves and boxes, so I have a structure that mirrors my physical space.

**Item Capture**
As a user, I want to photograph and label items and assign them to a specific box and shelf, so I have a visual record of what I own and where it lives.

**Remote Lookup**
As a user, I want to search for an item by name and see its exact location (unit, shelf, box), so I can decide whether I need to make a trip to the unit.

**QR Code Assignment**
As a user, I want to print a QR code and attach it to a box or shelf, so I can scan it later to instantly pull up everything stored there.

**Barcode Scanning**
As a user, I want to scan a product barcode to quickly log a familiar item, so I don't have to type everything manually.

**Multi-Unit Management**
As a user with two storage units, I want to see all my units in one app and switch between them easily, so I'm not juggling multiple tools or lists.

**Cloud Access**
As a user, I want my inventory to be available on any of my devices, so I never lose my data if I get a new phone.

---

## Assumptions

The following assumptions underpin the product direction and should be revisited as more is learned:

- **Users already rent storage.** The initial target market is existing storage unit renters, not people considering renting for the first time.
- **Users will invest in setup.** Users are willing to spend time cataloguing their items upfront in exchange for long-term convenience.
- **Cloud sync is a baseline expectation.** Users assume their data persists across devices and app reinstalls — local-only storage is not acceptable.
- **Photo + label is sufficient for v1.** Automatic AI object recognition is not required to deliver core value; users are comfortable labeling items themselves.
- **Item valuation is a future monetization or engagement lever.** Eventually, item photos should be usable for market value lookups, but this is not a v1 requirement.
- **The app will be cross-platform.** No strong preference for iOS or Android; a cross-platform framework (e.g., React Native or Flutter) is viable.

---

## Dependencies

| Dependency | Notes |
|---|---|
| Cloud storage / backend service | Required for data persistence and cross-device sync. Candidates include Firebase, Supabase, or AWS Amplify. Selection to be made during technical planning. |
| Camera & device permissions | App depends on OS-level camera and photo library access on both iOS and Android. Permission flows must be gracefully handled. |
| Barcode scanning library | A third-party library (e.g., MLKit, ZXing, or a React Native/Flutter equivalent) is needed for product barcode scanning functionality. |
| QR code generation | The system must generate unique QR codes assignable to any data element (unit, shelf, box, or item). A QR generation library and a print-friendly output format are required. |
| App store approval | Distribution via Apple App Store and Google Play requires compliance with each platform's review guidelines, which may affect timelines. |
