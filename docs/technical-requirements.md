# Technical Requirements: Merchandise Condition Photo Upload App

## 1. Overview

A cross-platform mobile application enabling company staff to document the current condition of merchandise in stores by capturing and uploading geotagged, timestamped photos. Staff self-register via the app and must be approved by an Application Owner or Admin before gaining access. The system includes a mobile client and a backend API with secure data storage.

-----

## 2. Goals & Objectives

- Allow staff to self-register using their Google account and await account approval.
- Allow Application Owners and Admins to activate or deactivate staff accounts.
- Allow any active staff member to select or add a store and upload photos of merchandise.
- Capture geolocation data at the time of photo capture.
- Timestamp all photos on both the client and server side.
- Provide a backend API to store and associate submissions with the correct Store and Staff member.

-----

## 3. Stakeholders

|Role             |Responsibility                                           |
|-----------------|---------------------------------------------------------|
|Field Staff      |Self-register, add stores, capture and upload photos     |
|Application Owner|Approve, activate, and deactivate staff accounts         |
|Admin            |Full system management — users, stores, and configuration|
|IT / DevOps      |Maintain infrastructure and deployment                   |

-----

## 4. Mobile Application Requirements

### 4.1 Platform Support

- **iOS**: iOS 15 and above
- **Android**: Android 10 (API Level 29) and above
- **Framework**: React Native (Expo managed workflow)

### 4.2 Registration Flow

- Staff register via the app using Google SSO (OAuth 2.0) using their personal Gmail account (`@gmail.com`)
- On first login with an unrecognised Google account, the app creates a pending registration
- Staff are shown a “Your account is awaiting approval” screen and cannot access app features until approved
- Staff are notified (push notification or email) when their account is approved or deactivated
- Re-registration with the same Google account is not permitted — a deactivated account must be reactivated by an Application Owner or Admin
- There is **no email domain restriction**; access control is enforced entirely through the approval workflow

### 4.3 Authentication

- Google SSO via OAuth 2.0
- Any valid Google/Gmail account may attempt registration — there is no domain restriction
- Access is controlled exclusively by account status (`pending`, `active`, `deactivated`), not by email domain
- JWT issued by the backend only if the account status is **active**; pending and deactivated accounts receive a clear error response with their current status
- Tokens stored securely using device keychain/keystore
- Session expiry with silent refresh; user prompted to re-authenticate when token cannot be refreshed
- If a session is active but the account is deactivated mid-session, the next API call returns a 403 and the app logs the user out with an explanatory message

### 4.4 Store Selection & Addition

- Any active staff member can upload to any store — no store assignment restrictions
- Staff must select a store before uploading photos
- Store list retrieved from the API (not hardcoded)
- Stores are identified by a system-generated UUID — store names are not unique
- Each store displays its full address alongside the name to help staff distinguish between stores with the same name
- Staff can search and filter stores by name or address
- Staff can optionally filter stores by proximity using device location
- **Staff can add a new store** if it does not already exist — they must provide store name and full address; the new store is immediately available for selection by all staff
- Selected store persisted during the session

### 4.5 Photo Capture & Upload

- Staff can capture photos using the native device camera or select from the gallery
- No limit on the number of photos per submission
- Each photo must capture:
  - **Timestamp**: ISO 8601 format, recorded at time of capture (device local time + UTC offset)
  - **Geolocation**: Latitude, longitude, and accuracy radius captured at time of photo
  - **Store ID**: As selected by the staff member
  - **Staff ID**: Derived from the authenticated session
- Photos should be compressed before upload to reduce bandwidth (target: max 1–2 MB per image)
- Support batch upload of multiple photos per session
- Upload progress indicator shown to the user
- Retry mechanism for failed uploads (with offline queue)

### 4.6 Offline Support

- Photos taken without connectivity are queued locally
- Upload occurs automatically when connectivity is restored
- User notified of pending uploads

### 4.7 Permissions Required

- Camera
- Photo Library / Media Storage
- Location (Fine Location — required at time of capture)
- Internet Access
- Push Notifications (for account approval/deactivation alerts)

-----

## 5. Account Management Requirements

### 5.1 Overview

Account management is available **directly within the mobile app** for both Application Owners and Admins. There is no separate admin web interface required for this functionality in v1. Application Owners and Admins log in with the same Google SSO flow as staff but are presented with an additional **Account Management** section in the app based on their role.

### 5.2 Application Owner Capabilities (In-App)

- View list of all staff accounts with their current status: **Pending**, **Active**, or **Deactivated**
- Approve (activate) a pending staff registration
- Deactivate an active staff account
- Reactivate a deactivated staff account
- View basic staff profile: name, email, registration date, last active date
- Cannot manage other Application Owner or Admin accounts

### 5.3 Admin Capabilities (In-App)

- All Application Owner capabilities above
- Manage Application Owner accounts (activate, deactivate, reactivate)
- View full audit log of account status changes
- Update store records
- Assign or change user roles (staff ↔ application_owner)

### 5.4 Account Status Lifecycle

```
[New Registration] → PENDING → (Approved by Application Owner/Admin) → ACTIVE
                                                              ↕
                                              (Deactivated by Application Owner/Admin) → DEACTIVATED
                                              (Reactivated by Application Owner/Admin) → ACTIVE
```

### 5.5 Business Rules

- A staff member with **Pending** or **Deactivated** status cannot log in or use the app
- Application Owners can only manage accounts at the staff role level — they cannot create or modify other Application Owner or Admin accounts
- Admins have full account management access across all roles
- Account status changes must be logged with a timestamp and the actor who made the change

-----

## 6. Backend API Requirements

### 6.1 Technology Stack (Recommended)

- **Runtime**: Node.js (Express or Fastify)
- **Database**: PostgreSQL (relational data) + DigitalOcean Spaces (photo storage)
- **Authentication**: Google OAuth 2.0 token verification + JWT issuance
- **Hosting**: DigitalOcean App Platform

### 6.2 Authentication & Registration Endpoints

|Method|Endpoint       |Description                                                                                                                                                                              |
|------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|POST  |`/auth/google` |Accepts Google ID token; if new account, creates a pending registration and returns `account_pending` status; if active, returns JWT; if deactivated, returns `account_deactivated` error|
|POST  |`/auth/refresh`|Accepts refresh token, returns new access token (only for active accounts)                                                                                                               |
|POST  |`/auth/logout` |Invalidates refresh token                                                                                                                                                                |

### 6.3 User / Account Management Endpoints

|Method|Endpoint           |Description                                                                       |
|------|-------------------|----------------------------------------------------------------------------------|
|GET   |`/users`           |Application Owner/Admin only — returns paginated list of staff with account status|
|GET   |`/users/:id`       |Application Owner/Admin only — returns staff profile and status                   |
|PATCH |`/users/:id/status`|Application Owner/Admin only — sets account status to `active` or `deactivated`   |
|GET   |`/users/me`        |Returns the authenticated staff member’s own profile                              |

### 6.4 Store Endpoints

|Method|Endpoint     |Description                                                                       |
|------|-------------|----------------------------------------------------------------------------------|
|GET   |`/stores`    |Any active staff — returns full list of stores; supports search by name or address|
|GET   |`/stores/:id`|Any active staff — returns store details including full address                   |
|POST  |`/stores`    |Any active staff — creates a new store record with name and address               |
|PUT   |`/stores/:id`|Admin only — updates store details                                                |

### 6.5 Submission & Photo Endpoints

|Method|Endpoint                 |Description                                                                                  |
|------|-------------------------|---------------------------------------------------------------------------------------------|
|POST  |`/submissions`           |Creates a new submission record (store + staff + metadata)                                   |
|POST  |`/submissions/:id/photos`|Uploads one or more photos to an existing submission; no limit on photo count                |
|GET   |`/submissions`           |Returns paginated list of submissions; staff see their own; application owners/admins see all|
|GET   |`/submissions/:id`       |Returns a specific submission with photo metadata                                            |

### 6.6 Photo Metadata Captured by API

Each photo record must store:

|Field              |Type    |Description                                |
|-------------------|--------|-------------------------------------------|
|`photo_id`         |UUID    |Unique identifier                          |
|`submission_id`    |UUID    |Parent submission                          |
|`staff_id`         |UUID    |Uploader identity                          |
|`store_id`         |UUID    |Selected store                             |
|`file_url`         |String  |Secure pre-signed URL to stored file       |
|`client_timestamp` |DateTime|Timestamp from the device at capture       |
|`server_timestamp` |DateTime|Timestamp recorded by the server on receipt|
|`latitude`         |Decimal |GPS latitude at time of capture            |
|`longitude`        |Decimal |GPS longitude at time of capture           |
|`location_accuracy`|Decimal |GPS accuracy in metres                     |
|`mime_type`        |String  |e.g., `image/jpeg`                         |
|`file_size_bytes`  |Integer |File size in bytes                         |

-----

## 7. Data Model (Simplified)

```
Staff
  - staff_id (PK, UUID)
  - name
  - email                   -- Google account email, unique
  - role                    -- enum: staff | application_owner | admin
  - account_status          -- enum: pending | active | deactivated
  - registered_at
  - last_active_at
  - approved_by (FK → Staff, nullable)
  - approved_at (nullable)

AccountStatusLog
  - log_id (PK, UUID)
  - staff_id (FK → Staff)   -- account being changed
  - changed_by (FK → Staff) -- application owner/admin who made the change
  - from_status
  - to_status
  - changed_at

Store
  - store_id (PK, UUID)
  - name                    -- not unique; same name may exist across locations
  - address_line_1
  - address_line_2 (optional)
  - city
  - state_province
  - postcode
  - country
  - latitude                -- derived/geocoded from address
  - longitude               -- derived/geocoded from address
  - created_by (FK → Staff)
  - created_at

Submission
  - submission_id (PK, UUID)
  - staff_id (FK → Staff)
  - store_id (FK → Store)
  - created_at

Photo
  - photo_id (PK, UUID)
  - submission_id (FK → Submission)
  - file_url
  - client_timestamp
  - server_timestamp
  - latitude
  - longitude
  - location_accuracy
  - file_size_bytes
  - mime_type
```

-----

## 8. Security Requirements

- All API communication over HTTPS (TLS 1.2+)
- Google ID tokens validated against Google’s public keys (not trusted blindly)
- JWTs signed with RS256; access token expiry: 15 minutes; refresh token expiry: 7 days
- No email domain restriction on Google SSO — any Gmail account may register; access is gated entirely by the Application Owner/Admin approval workflow
- Account status checked on every authenticated request — deactivated accounts receive 403 immediately
- Photo URLs must be pre-signed / time-limited (not publicly accessible)
- Role-based access control (RBAC):
  - **Staff**: upload to any store, add stores, view own submissions
  - **Application Owner**: all of the above + approve/activate/deactivate staff accounts
  - **Admin**: full access including store updates and application owner account management
- All account status changes are audit-logged with actor and timestamp
- All PII and location data stored in compliance with applicable privacy regulations (e.g., Privacy Act if in AU)

-----

## 9. Non-Functional Requirements

|Requirement                      |Target                                                             |
|---------------------------------|-------------------------------------------------------------------|
|Photo upload response time       |< 5 seconds per photo under normal 4G conditions                   |
|API availability                 |99.5% uptime                                                       |
|Max photo size (post-compression)|2 MB                                                               |
|Photos per submission            |No limit                                                           |
|Supported concurrent users       |500 (initial)                                                      |
|Photo storage retention          |Configurable; default 12 months                                    |
|Audit logging                    |All upload, authentication, and account status change events logged|

-----

## 10. Out of Scope (Initial Release)

- Application Owner / Admin review portal for viewing photo submissions (planned as a separate web-based application)
- Account management is **in scope** for the mobile app — the separate portal covers submission review only
- In-app photo annotation or editing
- Real-time notifications beyond account approval/deactivation alerts
- AI-based photo analysis (e.g., shelf condition detection)
- Integration with inventory management systems
- Store assignment or restriction by staff role

-----

## 11. Assumptions & Dependencies

- Staff use personal Gmail accounts (`@gmail.com`) for authentication — the company does not provide corporate Google Workspace accounts to field staff
- The first Admin account is provisioned manually by IT during initial setup (no self-service admin registration)
- Store names are not unique — the full address is the definitive disambiguator; UUID is the primary key for all store references
- Staff devices have GPS and camera hardware
- Network connectivity (3G/4G/Wi-Fi) is generally available at store locations, but offline queuing handles intermittent connectivity
- Cloud infrastructure provider is DigitalOcean

-----

## 12. Suggested Technology Stack Summary

|Layer         |Technology                                           |
|--------------|-----------------------------------------------------|
|Mobile App    |React Native (Expo managed workflow)                 |
|Authentication|Google OAuth 2.0 + JWT                               |
|Backend API   |Node.js + Express / Fastify                          |
|Database      |PostgreSQL (DigitalOcean Managed PostgreSQL)         |
|File Storage  |DigitalOcean Spaces (S3-compatible object storage)   |
|Hosting       |DigitalOcean App Platform                            |
|CI/CD         |GitHub Actions                                       |
|Monitoring    |DigitalOcean Monitoring + Papertrail (log management)|

-----

## 13. Estimated Monthly Infrastructure Cost (DigitalOcean)

> **Note:** Prices are based on current DigitalOcean published rates as of early 2026. Actual costs may vary with usage. Two scenarios are provided — a lean **Starter** setup for early-stage / low user volume, and a **Growth** setup for a more established user base with higher photo upload volumes.

### Scenario A — Starter (up to ~50 active staff, low upload volume)

|Service          |DigitalOcean Product            |Spec                                     |Monthly Cost       |
|-----------------|--------------------------------|-----------------------------------------|-------------------|
|API Backend      |App Platform (Basic, shared CPU)|1 container, 512 MB RAM                  |~$5.00             |
|Database         |Managed PostgreSQL (Single Node)|1 GB RAM, 10 GB SSD                      |~$15.00            |
|Photo Storage    |Spaces (Object Storage)         |250 GiB storage + 1 TiB transfer included|$5.00              |
|Log Management   |Papertrail (via DO Marketplace) |Developer plan (50 MB/day)               |~$7.00             |
|Bandwidth overage|App Platform egress             |Minimal at low volume                    |~$0–2.00           |
|**Total**        |                                |                                         |**~$32–34 / month**|

### Scenario B — Growth (up to ~500 active staff, moderate upload volume)

|Service          |DigitalOcean Product                      |Spec                                          |Monthly Cost       |
|-----------------|------------------------------------------|----------------------------------------------|-------------------|
|API Backend      |App Platform (Professional, dedicated CPU)|1 container, 1 vCPU / 2 GB RAM                |~$25.00            |
|Database         |Managed PostgreSQL (Single Node)          |2 GB RAM, 25 GB SSD                           |~$30.00            |
|Photo Storage    |Spaces (Object Storage)                   |250 GiB base + ~500 GiB additional @ $0.02/GiB|~$15.00            |
|Log Management   |Papertrail                                |Fixa plan (1 GB/day logs)                     |~$18.00            |
|Bandwidth overage|App Platform egress                       |Moderate upload/download traffic              |~$5–10.00          |
|**Total**        |                                          |                                              |**~$93–98 / month**|

### Key Pricing Notes

- **Spaces base plan** ($5/month) includes 250 GiB storage and 1 TiB outbound transfer — sufficient for low volume. Additional storage is $0.02/GiB and additional transfer is $0.01/GiB.
- **Photo storage growth** is the most variable cost driver. If 100 staff each upload 10 photos/day at ~1.5 MB each, that’s ~45 GB/month of new data. After 12 months, total storage could reach ~540 GB, pushing Spaces costs to ~$15–20/month.
- **Managed PostgreSQL single node** does not include a standby (failover) node. For production with higher reliability requirements, add a standby node (doubles the DB cost).
- **App Platform Professional** tier is recommended for production — shared CPU (Basic) plans can be throttled under sustained load.
- **Papertrail** is a third-party log management tool available via the DigitalOcean Marketplace. An alternative is DigitalOcean’s native monitoring (included free) for basic metrics, with Papertrail added only if centralised log search is needed.
- All prices are in **USD**. DigitalOcean bills monthly with no lock-in contracts.

### Cost Scaling Guidance

|Monthly Active Staff          |Estimated Monthly Cost|
|------------------------------|----------------------|
|Up to 50                      |~$32–40               |
|Up to 200                     |~$55–70               |
|Up to 500                     |~$90–110              |
|500+ (with HA DB + auto-scale)|~$150–200+            |

-----

*Version 0.6 — Draft for Review*