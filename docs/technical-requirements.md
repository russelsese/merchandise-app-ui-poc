# MerchSnap — UI Requirements Document
### Based on HTML POC · v1.0

---

## 1. Overview

**MerchSnap** is a mobile-first web application (targeting React Native in production) that enables field staff to document the condition of merchandise in stores by capturing and uploading geotagged, timestamped photos. This document reflects the functional and visual requirements as defined by the HTML proof-of-concept (POC).

---

## 2. App Identity

| Property      | Value                              |
|---------------|------------------------------------|
| App Name      | MerchSnap                          |
| Tagline       | Merchandise condition documentation |
| Primary Color | `#4169e1` (Royal Blue)             |
| Font Stack    | `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` |
| Viewport      | Mobile-first, 390px canonical width |

---

## 3. User Roles & Account States

### 3.1 Roles

| Role                | Description                                                        |
|---------------------|--------------------------------------------------------------------|
| Field Staff         | Self-registers, captures and uploads photos, browses stores        |
| Application Owner   | Approves, activates, and deactivates staff accounts                |
| Admin               | Full system access including store edits and owner management      |

### 3.2 Account Status Lifecycle

```
[Google Sign-In] → PENDING → (Approved) → ACTIVE
                                               ↕
                               (Deactivated) → DEACTIVATED → (Reactivated) → ACTIVE
```

| Status      | App Access                                               |
|-------------|----------------------------------------------------------|
| Pending     | Awaiting Approval screen only; no features accessible    |
| Active      | Full access to Dashboard, Store Selection, Photo Capture, Calendar |
| Deactivated | Blocked; shown an error on sign-in                       |

---

## 4. Navigation & User Flow

```
Login
  ├── [New / Pending user]  → Awaiting Approval → (loop until approved)
  └── [Active user]         → Dashboard
                                  ├── Take Photo  → Photo Capture & Upload
                                  ├── Stores      → Store Selection
                                  │                     ├── [Select store] → Photo Capture & Upload
                                  │                     └── Add Store      → Store Selection
                                  └── Calendar    → Activity Calendar
                                                        └── [Tap store]   → Photo Sheet
                                                                └── [Tap photo] → Lightbox
```

---

## 5. Screen Requirements

---

### 5.1 Login

**File:** `login.html`

#### Layout
- Full-screen blue accent header (brand gradient) with:
  - Camera icon in a white rounded square
  - App name **MerchSnap** and tagline
- Card body below the header containing:
  - "Welcome back" heading and descriptive copy
  - Horizontal divider labelled **Sign in**
  - Google Sign-In button
  - Info callout explaining the approval requirement
  - Footer with Terms of Service and Privacy Policy text links

#### Behaviour
| Element               | Behaviour                                               |
|-----------------------|---------------------------------------------------------|
| Google Sign-In button | Scales to 0.97 on press (active state); navigates to `awaiting-approval.html` (mocked — no real OAuth in POC) |
| Info callout          | Static; informs user that new accounts require approval |

#### Copy
- Button label: **Continue with Google**
- Info note: *"New accounts require approval before accessing the app. You'll be notified once your account is activated."*

---

### 5.2 Awaiting Approval

**File:** `awaiting-approval.html`

#### Layout
- Top bar with **Sign out** link (left) and **MerchSnap** wordmark (right)
- Centred content:
  - Three concentric pulsing rings (blue, animated)
  - Clock icon in a filled blue circle at the centre
  - Heading: **Account pending approval**
  - Descriptive paragraph
  - 3-step progress tracker (vertical)
- Amber notification callout (bottom)
- **Sign out** button (bottom)

#### 3-Step Progress Tracker

| Step | Label                    | State            | Visual                    |
|------|--------------------------|------------------|---------------------------|
| 1    | Google sign-in complete  | Done             | Blue filled, checkmark    |
| 2    | Awaiting approval        | Active (current) | Amber filled, dot         |
| 3    | Access granted           | Pending          | Grey filled, number "3"   |

#### Animations
- Three rings pulse with a `scale(1.12)` animation at 2.4s duration with staggered delays (0s, 0.8s, 1.6s)

#### Navigation
| Element              | Destination     |
|----------------------|-----------------|
| Sign out (top)       | `login.html`    |
| Sign out (bottom)    | `login.html`    |

---

### 5.3 Dashboard

**File:** `dashboard.html`

#### Layout
- **Header** (blue gradient `#4169e1 → #3457c8`):
  - Greeting: *"Good day 👋"*, User name (e.g. **Russell**), Role badge (**Field Staff**)
  - Avatar: circular, initials (e.g. **RS**)
  - Two stat chips: **Stores today** and **Photos uploaded**
- **Quick Actions** section:
  - Full-width primary tile — **Take Photo** (blue gradient, camera icon, subtitle shows last visited store)
  - Two-column row: **Stores** (green icon tile) | **Calendar** (violet icon tile)
- **Recent Visits** section:
  - List of last 3 store visits (coloured avatar, store name, relative timestamp, photo count badge, chevron)
  - **View full history →** link row at bottom

#### Mock Data (POC)
| Field         | Value                  |
|---------------|------------------------|
| User name     | Russell                |
| Role          | Field Staff            |
| Avatar        | RS                     |
| Stores today  | 2                      |
| Photos today  | 6                      |

#### Recent Visits (mock)
| Store            | Time              | Photos |
|------------------|-------------------|--------|
| Woolworths Metro | Today · 2:34 PM   | 4      |
| Coles            | Today · 11:22 AM  | 2      |
| Target           | Yesterday · 3:15 PM | 6    |

#### Navigation
| Element               | Destination                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| Take Photo tile       | `photo-capture.html?name=Woolworths+Metro&address=123+George+St…`           |
| Stores tile           | `store-selection.html`                                                      |
| Calendar tile         | `calendar.html`                                                             |
| Recent visit rows     | `calendar.html`                                                             |
| View full history     | `calendar.html`                                                             |

#### Behaviour
- Action tiles scale to 0.97 on press
- Visit rows highlight on press
- Main content area is independently scrollable

---

### 5.4 Store Selection

**File:** `store-selection.html`

#### Layout
- Header with back button, title **Select a Store**, subtitle
- Search bar (magnifying glass icon, clear × button)
- Filter pills row: **All Stores** | **Near Me** · Dynamic store count (right-aligned)
- Scrollable store list:
  - Each row: coloured initial avatar, store name (bold), full address, optional distance badge, chevron
- Empty state: icon + *"No stores found"* + hint to add store
- FAB (bottom right): **+ Add Store** (blue pill button)

#### Search Behaviour
- Filters list in real-time as user types (name and address)
- Matching text is highlighted with a blue `<mark>` style
- Clear button (×) appears when input is non-empty
- Empty state shown when no stores match

#### Near Me Filter
- Toggles the active pill (blue → grey / grey → blue)
- Re-sorts list by ascending distance (km)
- Distance badge appears on each row when active

#### Store List (mock data — 7 stores)

| Store Name       | Address                              | Distance | Colour  |
|------------------|--------------------------------------|----------|---------|
| Woolworths Metro | 123 George St, Sydney NSW 2000       | 0.3 km   | Green   |
| Coles            | 12 Elizabeth St, Sydney NSW 2000     | 0.5 km   | Red     |
| Woolworths       | 456 Bourke St, Melbourne VIC 3000    | 1.2 km   | Green   |
| Target           | 45 Swanston St, Melbourne VIC 3000   | 1.8 km   | Amber   |
| Woolworths       | 789 Queen St, Brisbane QLD 4000      | 2.1 km   | Green   |
| Coles Central    | 288 Bourke St, Melbourne VIC 3000    | 3.4 km   | Red     |
| Big W            | 550 George St, Sydney NSW 2000       | 4.1 km   | Blue    |

#### Navigation
| Element          | Destination                                            |
|------------------|--------------------------------------------------------|
| Back button      | `dashboard.html`                                       |
| Store row tap    | `photo-capture.html?name={name}&address={address}`    |
| Add Store FAB    | `add-store.html`                                       |

---

### 5.5 Add New Store

**File:** `add-store.html`

#### Layout
- Header with back button, title **Add New Store**, subtitle
- Scrollable form:
  - Store Name *(required)*
  - Address Line 1 *(required)*
  - Address Line 2 *(optional)*
  - City / Suburb *(required, half-width)*
  - State *(required, half-width)*
  - Postcode *(required, half-width, numeric keyboard)*
  - Country *(required, half-width)*
  - Blue info callout: *"This store will be immediately available for all staff…"*
- Sticky bottom **Add Store** button

#### Form Validation
- All required fields validated on submit
- Fields with errors: red border + inline error message below
- Errors clear individually as user types in each field
- Page auto-scrolls to first errored field on submit

#### Success State
- Full-screen white overlay fades in (0.3s)
- Green animated checkmark (pop scale animation)
- **Store Added!** heading
- Confirmation of store name and address
- **Back to Store List** button

#### Navigation
| Element                   | Destination          |
|---------------------------|----------------------|
| Back button               | `store-selection.html` |
| Back to Store List button | `store-selection.html` |

---

### 5.6 Photo Capture & Upload

**File:** `photo-capture.html`

#### Layout
- Header: back button, **Upload Photos** title
- **Store banner**: store icon, store name and address, **Change** link
- **Geo/Timestamp strip**: live GPS coordinates (left) | live clock HH:MM:SS (right)
- **Content area** (toggles between two states):
  - **Empty state** (no photos): dashed camera icon box, heading, Take Photo button, Choose from Gallery button
  - **Photo grid** (photos selected): 3-column grid of thumbnails with × remove button on each, Camera + Gallery row buttons below
- **Bottom bar** (visible when photos selected): **Upload {n} Photos** button

#### Upload Overlay (triggered on upload)
- **Progress state**: upload icon, *"Uploading…"* heading, per-photo progress bar (0–100%), percentage counter
- **Success state**: animated green checkmark, *"Upload Complete!"*, confirmation text, **Upload More Photos** + **Change Store** buttons

#### Real Behaviours in POC
| Feature              | Implementation                                                        |
|----------------------|-----------------------------------------------------------------------|
| Camera input         | `<input type="file" accept="image/*" capture="environment">`         |
| Gallery input        | `<input type="file" accept="image/*" multiple>`                      |
| Photo preview        | FileReader API → `readAsDataURL()` → rendered in grid                |
| Remove photo         | Splices from array, re-renders grid                                   |
| Geolocation          | `navigator.geolocation.getCurrentPosition()` with fallback to `–33.8688, 151.2093` (Sydney) |
| Live clock           | `setInterval` every 1000ms updating time label                       |
| Upload simulation    | Per-photo progress animation using `requestAnimationFrame` + easeOut timing, ~1.2s per photo |
| Store name/address   | Read from URL query parameters (`?name=…&address=…`)                 |

#### Navigation
| Element                | Destination            |
|------------------------|------------------------|
| Back button            | `store-selection.html` |
| Change link            | `store-selection.html` |
| Upload More Photos     | Resets screen (clears photos) |
| Change Store           | `store-selection.html` |

---

### 5.7 Activity Calendar

**File:** `calendar.html`

#### Layout
- Header: back button, **Activity Calendar** title, **34 photos this week** summary badge
- Scrollable 7-day list:
  - Each day: date label (Today / Yesterday / Day, DD Mon), full date, total photo count
  - Days with no visits: grey empty-state row (*"No store visits"*)
  - Days with visits: list of store rows

#### Store Row (within each day)
- Coloured initial avatar, store name, full address, visit time, photo count (bold + "photos" label), chevron
- Tapping opens the **Photo Sheet**

#### Photo Sheet (bottom drawer)
- Slides up from bottom (cubic-bezier animation), semi-transparent backdrop
- Drag handle at top
- Store header: avatar, name, address, *"Visited at {time} · {n} photos"*
- Close (×) button
- 3-column photo grid with:
  - Lazy-loaded images (picsum.photos placeholder in POC)
  - Skeleton shimmer loader while loading
  - Tap to open **Lightbox**

#### Lightbox (full-screen photo viewer)
- Dark overlay (`rgba(0,0,0,0.95)`)
- Top bar: close (×) button (left), **{n} / {total}** counter (centre)
- Full-size image (`object-fit: contain`)
- Bottom bar: Previous (←) button, dot indicator (up to 7 dots, active dot wider), Next (→) button
- Boundary states: prev/next buttons dim to 0.3 opacity at first/last photo
- Swipe support: touch delta > 50px triggers navigation

#### Mock Data — 7 Days

| Day              | Stores Visited                                    | Photos |
|------------------|---------------------------------------------------|--------|
| Today (22 Feb)   | Woolworths Metro (4), Coles (2)                   | 6      |
| Yesterday (21 Feb) | Target (6), Big W (3)                           | 9      |
| Thu 20 Feb       | Woolworths Melbourne (5)                          | 5      |
| Wed 19 Feb       | Coles Central (8), Woolworths Metro (2)           | 10     |
| Tue 18 Feb       | *(no visits)*                                     | 0      |
| Mon 17 Feb       | Target (4)                                        | 4      |
| Sun 16 Feb       | Big W (7), Coles (3)                              | 10     |
| **Week total**   |                                                   | **34** |

#### Navigation
| Element               | Destination           |
|-----------------------|-----------------------|
| Back button           | `dashboard.html`      |
| Store row tap         | Opens Photo Sheet     |
| Backdrop tap          | Closes Photo Sheet    |
| Photo thumbnail tap   | Opens Lightbox        |
| Lightbox close (×)    | Closes Lightbox       |

---

## 6. Design System

### 6.1 Colour Palette

| Token          | Hex       | Usage                                      |
|----------------|-----------|--------------------------------------------|
| Brand 500      | `#4169e1` | Primary buttons, headers, active states    |
| Brand 600      | `#3457c8` | Gradient end, darker accents               |
| Brand 50       | `#f0f4ff` | Active nav backgrounds, info tints         |
| Brand 100      | `#dce8ff` | Highlight marks, light accents             |
| Green 600      | `#16a34a` | Woolworths avatar, Stores tile             |
| Red 600        | `#dc2626` | Coles avatar, error states                 |
| Amber 600      | `#d97706` | Target avatar, active step (approval)      |
| Blue 700       | `#1d4ed8` | Big W avatar                               |
| Violet 600     | `#7c3aed` | Calendar tile                              |
| Gray 100       | `#f3f4f6` | Input backgrounds, nav item backgrounds    |
| Gray 200       | `#e5e7eb` | Borders, dividers, skeleton base           |
| Gray 400       | `#9ca3af` | Placeholder text, secondary labels         |
| Gray 500       | `#6b7280` | Body text, subtitles                       |
| Gray 800       | `#1f2937` | Primary body text                          |

### 6.2 Typography

| Size   | Tailwind  | Usage                               |
|--------|-----------|-------------------------------------|
| 10px   | `text-xs` (tracking-widest) | Section labels (uppercase) |
| 12px   | `text-xs` | Secondary labels, badges, captions  |
| 13px   | `text-sm` | Nav descriptions, body copy         |
| 14px   | `text-sm` (font-medium/semibold) | Buttons, form labels |
| 16px   | `text-base`| Tile labels, form inputs           |
| 20px   | `text-xl` | Dashboard heading, screen titles    |
| 24px   | `text-2xl`| Login heading, approval heading     |

### 6.3 Border Radius

| Token     | Tailwind       | Usage                                   |
|-----------|----------------|-----------------------------------------|
| Large     | `rounded-2xl`  | Cards, store rows, form inputs, badges  |
| Extra large | `rounded-3xl` | Action tiles, primary buttons          |
| Full      | `rounded-full` | Pills, FAB, avatar circles             |

### 6.4 Interactive States

| State  | Behaviour                                 |
|--------|-------------------------------------------|
| Active | `transform: scale(0.95–0.97)` on press    |
| Hover  | Background lightens (`hover:bg-gray-50`)  |
| Focus  | Blue border + `box-shadow` ring on inputs |
| Error  | Red border (`#ef4444`) + inline message   |

### 6.5 Phone Shell (POC Navigator only)
- Dark body (`#1c1c1e`), border-radius `52px`, multi-layer box-shadow
- Dynamic Island (pill, `118×34px`) at top centre
- Side buttons: silent toggle, volume up/down (left), power (right)
- Home indicator bar (bottom centre)
- Scales to fit viewport height via JS `transform: scale()`

---

## 7. Shared Patterns

### 7.1 Phone Shell (standalone pages)
- `max-width: 390px`, `height: 100vh`, `overflow: hidden`
- Centred on desktop with `box-shadow` for phone effect
- Shadow removed on actual mobile (`≤390px` breakpoint)

### 7.2 Status Bar Spacer
- All screens include a `h-10` spacer at the top to simulate the iOS/Android status bar

### 7.3 Scrollable Content
- Scrollable areas use `flex: 1; overflow-y: auto; -webkit-overflow-scrolling: touch`
- Scrollbars hidden via `::-webkit-scrollbar { display: none }`

### 7.4 Empty States
- Consistent pattern: icon in rounded container + heading + supporting text
- Used in: Store Selection (no search results), Photo Capture (no photos selected), Calendar (no visits on a day)

### 7.5 Overlays
- Bottom sheet: `position: absolute`, `transform: translateY(100%)` → `translateY(0)`, cubic-bezier timing
- Lightbox / success overlays: `opacity: 0; pointer-events: none` → `opacity: 1; pointer-events: all`, 0.2–0.3s transition

---

## 8. File Structure

```
merchandise-app-ui-poc/
├── docs/
│   └── technical-requirements.md   ← this document
└── mock/  (or root — see current structure)
    ├── index.html                   ← POC screen navigator (not part of the app)
    ├── login.html                   ← Screen 1
    ├── awaiting-approval.html       ← Screen 2
    ├── dashboard.html               ← Screen 3
    ├── store-selection.html         ← Screen 4
    ├── add-store.html               ← Screen 5
    ├── photo-capture.html           ← Screen 6
    └── calendar.html                ← Screen 7
```

---

## 9. POC Scope & Limitations

The following are mocked or simplified in the HTML POC and will require real implementation:

| Feature                    | POC Behaviour                               | Production Requirement               |
|----------------------------|---------------------------------------------|---------------------------------------|
| Google Sign-In             | Button navigates directly (no OAuth)        | Google OAuth 2.0 via Expo Auth Session |
| Account approval           | Static screen (no polling)                  | Push notification / real-time status check |
| Store list                 | Hardcoded array of 7 stores                 | API: `GET /stores`                    |
| Near Me distance           | Hardcoded distance values                   | Geolocation API + server-side distance sort |
| Add Store                  | Success overlay only (no persistence)       | API: `POST /stores`                   |
| Photo upload               | Simulated progress bar (~1.2s per photo)    | Multipart upload to `POST /submissions/:id/photos` |
| Geolocation                | `getCurrentPosition()` with Sydney fallback | Required; fail gracefully              |
| Calendar photos            | picsum.photos placeholder images            | Pre-signed URLs from DigitalOcean Spaces |
| Calendar history           | Hardcoded 7-day dataset                     | API: `GET /submissions?staff_id=me`   |
| User profile               | Hardcoded "Russell / Field Staff"           | JWT claims from authenticated session  |
| Account management screens | Not built in POC                            | In-app for Application Owner / Admin  |

---

## 10. Out of Scope (v1 POC)

- Account management screens (Application Owner / Admin role)
- Push notification delivery
- Offline photo queue
- Real authentication and session management
- Backend API and database
- Admin/Owner web portal for submission review

---

*Document generated from HTML POC review · MerchSnap v1.0 POC · February 2026*
