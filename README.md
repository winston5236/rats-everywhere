Let's pull together everything currently in the project outputs to make sure the handoff doc is accurate:# Project Handoff Summary — "le rat is in da house"

## What this project is

A two-part system for simulating rat detection/deterrence in a home, with an optional real-world room scan feeding into it:

1. **`index.html`** — a self-contained, single-file 3D web app (Three.js) simulating a 36×36×36-unit grid representing a home. It includes: a rat with pathfinding AI and erratic wandering behavior, an editable furniture/wall/human/avoidance-zone system with drag-and-drop placement, rotation, and drag-to-resize, a predator-sound deterrence system (owl/kite/cat/snake, each with distinct wave shapes and effectiveness), a procedural "Random Stimulation" home floor-plan generator, a localStorage-based account/save system, and a **Scan tab** that can import a real scanned room and rebuild it using the site's own furniture/wall objects (not just a static mesh).

2. **iOS companion app** (`RoomScanCaptureView.swift`) — uses Apple's RoomPlan (LiDAR) to scan a real room, saves the result locally on-device (no server, nothing uploaded anywhere), and can open `index.html` in an embedded in-app browser (`WKWebView`), automatically feeding the scan data into it via a JS bridge — so scanning and viewing the resulting 3D layout both happen inside one native app.

Everything is designed to be fully local — no backend server exists anywhere in this project.

---

## Current status

- **The web app (`index.html`) works standalone** — this part has been tested and iterated on extensively and is solid on its own (open it in any browser, or serve it locally).
- **The iOS capture screen itself is believed working** — RoomPlan scanning, live wall/object counts, local save, and the Share Sheet export have all been built and reasoned through carefully, though not confirmed against real device output (see Known Issues).
- **The in-app "View in Rat Monitor" WKWebView screen is currently blank when opened** — this is the open bug. See below.

---

## Known issues / open items

### 1. Blank screen when tapping "🐀 View in Rat Monitor" (unresolved)
We were mid-diagnosis when this handoff was requested. What's been done:
- Added a visible on-screen error banner + Reload button to that screen (previously, any failure just silently showed nothing — this was itself a bug, now fixed).
- Enabled `webView.isInspectable = true` so Safari's remote Web Inspector can attach to the WKWebView specifically (iOS 16.4+ only).
- **The most recent Xcode screenshot shows a `?` badge on the `1. index` file in the project navigator** — this is the same "broken reference" symptom seen earlier with a stray `Info.plist` reference in this project. **This is the top suspect** and should be checked first: right-click `1. index` → confirm it resolves to a real file, and confirm it's checked under the app target's **Target Membership**, and appears in **Build Phases → Copy Bundle Resources**.
- The Xcode console log shown (`FigCaptureSourceRemote` asserts, sandbox extension warnings) is very likely generic system/media-pipeline noise unrelated to the actual bug — Xcode's console mixes in a lot of OS-level chatter. It's not yet been confirmed whether this was run on a real LiDAR device or the Simulator — **RoomPlan requires a real LiDAR device (iPhone 12 Pro+ or iPad Pro) and does not work in the Simulator at all.**
- **Next diagnostic step, if the `?` badge isn't the answer:** run on a real device with the device plugged into a Mac, then Safari (Mac) → Develop → [device name] → [the page] → Console tab, and read the actual JS error.

### 2. RoomPlan JSON shape was never verified against real output
The parsing logic in `index.html`'s Scan tab (field names like `walls`/`doors`/`objects`, the `transform` array layout, `category` string values) was written from documentation/best understanding of Apple's `CapturedRoom` Codable structure, **not confirmed against an actual exported `.json` file**. If a real scan produces 0 objects or wrong shapes, the JSON's actual structure needs to be inspected and the parser adjusted to match reality.

### 3. Furniture-category mapping is incomplete
`ROOMPLAN_TYPE_MAP` in `index.html` only maps a subset of RoomPlan's object categories (bed, sofa, chair, table, refrigerator→fridge, storage→cabinet, stove/oven/sink→counter) to the site's furniture types. Categories like toilet, bathtub, stairs, fireplace, television, dishwasher, washerDryer are intentionally skipped (no equivalent furniture type exists in the site) — this is a deliberate simplification, not a bug, but worth knowing about.

---

## Complete file list

| File | Purpose | Status |
|---|---|---|
| `index.html` | The full 3D rat simulation web app. Single self-contained file. | ✅ Current, working standalone |
| `roomscan/ios/RoomScanCaptureView.swift` | iOS RoomPlan capture screen + embedded WKWebView viewer + JS bridge. Drop into an Xcode project. | ⚠️ Blank-screen bug being diagnosed |
| `roomscan/convert/convert_usdz_to_glb.py` | Optional local Blender script to convert `.usdz` → `.glb`, only needed if using the site's "overlay raw mesh" option (not the main furniture-rebuild path). | ✅ Not part of the current bug |
| `roomscan/README.md` | Explains the local-only capture → convert → import pipeline. | ⚠️ Slightly stale — written before the in-app WKWebView flow existed; still accurate for the manual/desktop-browser workflow |
| `roomscan/web/scan-viewer-demo.html` | An old standalone scan-mesh viewer page, from before scan import was built directly into `index.html`. | 🗑️ Obsolete — safe to ignore/delete |
| `roomscan/web/integration-snippet.js` | Old copy-paste starter code for merging scan import into a site — superseded by it now being built directly into `index.html`'s Scan tab. | 🗑️ Obsolete — safe to ignore/delete |
| `README.md` (top-level) | Generic GitHub Pages deployment instructions for `index.html` alone (predates the iOS integration). | ✅ Still accurate for web-only deployment |

**Not part of the deliverables, but referenced in-app:** the Xcode project itself (`le_rat_is_in_da_houseApp`, `Assets`) — those are the teammate's own project scaffolding, not something generated here.

---

## Everything the teammate needs on their end

**Files to receive:**
- `index.html`
- `roomscan/ios/RoomScanCaptureView.swift`
- (optional) `roomscan/convert/convert_usdz_to_glb.py` + `roomscan/README.md`
- Ignore/discard `roomscan/web/scan-viewer-demo.html` and `roomscan/web/integration-snippet.js`

**Hardware requirement:**
- A physical iPhone 12 Pro/Pro Max or later Pro model, or an iPad Pro with LiDAR — **the Simulator cannot be used** for any of the RoomPlan scanning functionality.

**Xcode project setup checklist:**
1. Drag `RoomScanCaptureView.swift` into the project (target membership checked).
2. Drag `index.html` into the project **as a resource** — confirm it shows up under **Build Phases → Copy Bundle Resources** (this is the #1 suspect for the current bug — check this first).
3. Target → **Info** tab → add these three custom properties (there is no physical `Info.plist` file in this project; it's auto-generated, so these are added via the UI, not by editing a file):
   - `Privacy - Camera Usage Description` → e.g. "Used to scan your room in 3D."
   - `Application supports iTunes file sharing` → `YES`
   - `Supports opening documents in place` → `YES`
4. Target → **General** → **Minimum Deployments** → iOS 16.0 or higher (RoomPlan requires it; `isInspectable` requires iOS 16.4+ specifically to be usable).
5. Target → **Signing & Capabilities** → set a Team (a free Apple ID is enough to run on your own device).
6. Build and run on the physical device, trusting the developer certificate on-device the first time if prompted (Settings → General → VPN & Device Management).

**To give a fresh Claude conversation full context fast:** paste this entire summary in as the first message, along with the two files above — that's enough for a new conversation (with either of us) to pick up exactly where this one left off without re-deriving the architecture.
