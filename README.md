# MerchSnap — UI Proof of Concept

A mobile-first HTML proof-of-concept for **MerchSnap**, a field merchandising app that enables staff to document store conditions by capturing and uploading geotagged, timestamped photos.

## Live Demo

**[View the POC → https://russelsese.github.io/merchandise-app-ui-poc/](https://russelsese.github.io/merchandise-app-ui-poc/)**

> Open the link above on a mobile device or use your browser's DevTools to emulate a mobile screen (390px width recommended) for the best experience.

---

## Screens

| Screen | File | Description |
|--------|------|-------------|
| POC Navigator | `index.html` | Jump between all screens from one place |
| Login | `login.html` | Google Sign-In entry point |
| Awaiting Approval | `awaiting-approval.html` | Pending account status with animated progress tracker |
| Dashboard | `dashboard.html` | Home screen with quick actions and recent visits |
| Store Selection | `store-selection.html` | Searchable and filterable store list |
| Add New Store | `add-store.html` | Form to add a new store with validation |
| Photo Capture & Upload | `photo-capture.html` | Camera/gallery picker with simulated upload progress |
| Activity Calendar | `calendar.html` | 7-day visit history with photo sheet and lightbox |

---

## User Flow

```
Login
  ├── New / Pending user  → Awaiting Approval
  └── Active user         → Dashboard
                                ├── Take Photo  → Photo Capture & Upload
                                ├── Stores      → Store Selection
                                │                     ├── Select store → Photo Capture & Upload
                                │                     └── Add Store    → Add New Store
                                └── Calendar    → Activity Calendar
                                                      └── Tap store   → Photo Sheet → Lightbox
```

---

## Tech Stack

- Plain HTML, CSS, and vanilla JavaScript — no build step required
- [Tailwind CSS](https://tailwindcss.com/) via CDN for utility styling
- [Lucide Icons](https://lucide.dev/) via CDN
- Native browser APIs: `FileReader`, `Geolocation`, `requestAnimationFrame`

---

## Running Locally

No build step or server required. Simply open any `.html` file in a browser:

```bash
# Clone the repo
git clone https://github.com/russelsese/merchandise-app-ui-poc.git
cd merchandise-app-ui-poc

# Open the POC navigator
open index.html   # macOS
start index.html  # Windows
```

Or start a simple local server for best results:

```bash
npx serve .
# then open http://localhost:3000
```

---

## POC Scope & Limitations

This is a static HTML prototype. The following are mocked and will require real implementation in production:

| Feature | POC Behaviour |
|---------|--------------|
| Google Sign-In | Button navigates directly — no real OAuth |
| Account approval | Static screen — no real-time polling |
| Store list | Hardcoded 7 stores |
| Photo upload | Simulated progress bar (~1.2s per photo) |
| Geolocation | `getCurrentPosition()` with Sydney fallback |
| Calendar photos | Placeholder images from picsum.photos |
| User profile | Hardcoded "Russell / Field Staff" |

See [docs/requirements.md](docs/requirements.md) for the full UI requirements document.

---

## User Roles

| Role | Description |
|------|-------------|
| Field Staff | Captures and uploads store photos |
| Application Owner | Approves and manages staff accounts |
| Admin | Full system access |

---

*MerchSnap v1.0 POC · February 2026*
