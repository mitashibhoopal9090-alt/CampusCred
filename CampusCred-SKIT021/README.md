# CampusCred — Digital Student Achievement & Participation Record System

**Problem Statement Code:** SKIT021
**Tagline:** *Every Achievement Counts. Every Record Verified.*

CampusCred is a centralized college platform where students submit achievements and participation
records with certificate proof, authorized faculty verify them, and admins manage college-wide
records, view analytics and download reports.

> **Frontend prototype.** This repository contains a complete, clickable, multi-page frontend built
> with HTML5, CSS3 and vanilla JavaScript (no build step, no framework, no network dependency).
> All data is fictional demonstration data. See *Backend Integration Readiness* below.

---

## 1. How to run locally

No installation or build is required — it is a static site.

### Option A — just open it
Double-click `index.html` (or open it in Chrome/Edge/Firefox). The app runs entirely from the file system.

### Option B — local web server (recommended for a clean URL bar)

```bash
# Python 3 (bundled with most systems)
python -m http.server 8080
# then open http://localhost:8080

# …or Node.js
npx serve .
# …or VS Code "Live Server" extension → Open with Live Server
```

### Demo logins

| Role | Email | Password |
|---|---|---|
| **Student** | `aarav.sharma@campuscred.demo` | `student123` |
| **Faculty / Verifier** | `rohit.mehta@campuscred.demo` | `faculty123` |
| **Admin** | `admin@campuscred.demo` | `admin123` |

Enrollment ID also works for the student login (e.g. `2023CS041`).
The login page has **one-click demo buttons** that fill these credentials for you.

### Pages that need no login

- `#/` — Landing page
- `#/login` — Login (add `?role=faculty` or `?role=admin` to preselect a role)
- `#/verify` — Public certificate verification (`#/verify/CC-2026-0140` opens a specific record)
- `#/portfolio/STU-1001` — Public verified portfolio

---

## 2. Project structure

```
index.html                  App shell + Firebase SDKs + script order
css/
  base.css                  Design tokens, reset, typography, forms, buttons
  layout.css                Sidebar / topbar shell, responsive rules
  components.css            Cards, tables, badges, modals, toasts, charts, tabs
  pages.css                 Landing, login, dashboards, portfolio, resume, verify
js/
  data/mockData.js          ← Single source of initial seed records
  services/                 ← Google Cloud Firestore & Service layer
    firebaseConfig.js       Firebase SDK initialization & credentials manager
    firestoreService.js     Google Cloud Firestore 13-collection service & seeder
    store.js                Data store + dual Firestore/localStorage cache & multi-session
    authService.js          Multi-user login, account switcher, user directory, register
    activityService.js      CRUD, shared query/filters, notifications, remarks
    verificationService.js  Approve/reject, queues, audit trail, certificate IDs
    profileService.js       Profile, social links, public portfolio projection
    certificateService.js   Certificate lookup + QR renderer + SHA-256 signatures
    reportService.js        Aggregations, charts series, CSV/PDF export
  components/
    icons.js                SVG icon set + CampusCred logo
    ui.js                   Toasts, modals, confirm, tables, empty/loading states
    charts.js               Dependency-free SVG charts (bar/stacked/line/donut/hbar)
    shell.js                Role-based sidebar, topbar + Multi-Account Switcher & DB console
  pages/
    landing.js  login.js  student.js  faculty.js  admin.js  public.js
  app.js                    Hash router, auth guards, boot
```

---

## 3. Firestore Database Architecture (13 Collections)

The application is connected to Google Cloud Firestore with a normalized entity model and zero-join read caches:

1. **`users`** — Accounts matching Auth UIDs (`uid`, `name`, `email`, `role`, `departmentId`, `departmentName`, `courseId`, `courseName`, `academicYear`, `accountStatus`)
2. **`studentProfiles`** — Student academic identity (`uid`, `bio`, `headline`, `registrationNumber`, `currentSemester`, `isPortfolioPublic`, `portfolioSlug`, `totalVerifiedAchievements`)
3. **`facultyProfiles`** — Verifier scope (`uid`, `designation`, `departmentId`, `authorizedCategories`, `officeLocation`, `totalVerifiedCount`)
4. **`departments`** — Institutional academic departments (`id`, `name`, `code`, `hodName`, `contactEmail`, `isActive`)
5. **`courses`** — Academic programs (`id`, `departmentId`, `name`, `code`, `durationYears`, `degreeType`, `isActive`)
6. **`activityCategories`** — Taxonomies (`id`, `name`, `description`, `icon`, `weightPoints`, `isActive`, `displayOrder`)
7. **`activities`** — Participation records, proof files, verifications (`id`, `studentUid`, `studentName`, `title`, `categoryId`, `proofDocument`, `verificationStatus`, `certificateId`, etc.)
8. **`verificationHistory`** — Immutable decision audit log (`id`, `activityId`, `studentUid`, `verifierUid`, `action`, `previousStatus`, `newStatus`, `remarks`, `timestamp`)
9. **`studentSkills`**, **`studentProjects`**, **`studentLinks`** — Resume & portfolio components
10. **`resumeSettings`** — Student resume preferences and selected verified activities
11. **`certificates`** — Verification anchors with SHA-256 signatures and QR endpoints
12. **`auditLogs`** — Administrative audit trail
13. **`notifications`** — In-app notification messages

---

## 4. Multiple User Login & Account Switcher

- **Quick User Directory**: 1-click login and role switching across any Student, Faculty, or Admin account.
- **Concurrent Multi-Account Sessions**: Stay signed in as multiple users simultaneously.
- **Top Bar Account Switcher**: Switch active user personas (Student ↔ Faculty ↔ Admin) instantly without entering passwords repeatedly.
- **User Registration**: Register new Students, Faculty, or Admins directly into Firestore collections.

---

## 3. Feature map

### Landing page
Hero ("Your Achievements. One Verified Digital Identity."), problem statement,
Get Started / Explore Platform CTAs, 6 feature cards, 4-step workflow
(Submit → Upload Proof → Faculty Verification → Verified Portfolio),
role cards, footer with **SKIT021** and the project name.

### Student
- **Dashboard** — welcome banner, stat cards (Total/Verified/Pending/Rejected), recent activity
  table with search + category + status filters, quick actions, notifications, faculty remarks
- **Add Activity** — full form (name, category, organization, event date, role, result,
  description, certificate upload with type/size validation + preview, optional certificate ID),
  required-field validation, success screen with reference ID, status = *Pending*
- **My Activities** — filterable table, detail modal with status timeline, delete pending,
  CSV export
- **My Portfolio** — verified achievements grouped by category with verification stamps,
  separate Pending/Rejected sections, social links marked *student-added, unverified*,
  copyable public link
- **Resume Builder** — ATS-Friendly / Modern / Academic CV templates, section checkboxes,
  editable summary + skills + project descriptions, live preview, **Verified by CampusCred**
  labels, print-to-PDF download
- **Social Links** — add/edit/delete + per-link Public/Private toggle
- **Notifications** — tabs (all/unread/approved/rejected/pending), mark-as-read
- **Profile** — academic details, portfolio visibility toggle, demo-data reset

### Faculty / Verifier
- **Dashboard** — pending/approved/rejected/total cards, queue preview, decision timeline,
  decisions-by-category chart
- **Pending Verification** — filters (category, department, status, date range), student +
  activity detail, certificate preview panel, **Approve / Reject with mandatory rejection
  reason**, confirmation modals, immediate status update
- **Reviewed Submissions**, **Verification History** (audit trail), **Profile**

### Admin
- **Dashboard** — 5 headline stats, category donut, monthly stacked bar, department ranking,
  recent submissions, recent verification feed, advanced filters
  (department, branch, academic year, category, status, date range) that re-render every chart
- **Students / Faculty / Activities** management grids + tables with search and filters
- **Analytics** — category chart, status donut, monthly trend line, department bars,
  top departments, department-wise student participation, searchable records table
- **Reports** — 6 downloadable reports (CSV via Blob, PDF via print stylesheet) + preview table
- **Certificate Verification** — manual ID search + simulated QR scan + sample ID loader
- **Settings** — demo controls, backend integration notes, privacy defaults

### Public
- **Certificate verification** — VERIFIED / NOT FOUND results with full certificate details,
  QR deep links, honest "mock records only" disclaimers
- **Public portfolio** — verified achievements only; links carry no verification badge;
  private portfolios are hidden; no contact details or credentials exposed

---

## 4. Backend integration readiness (Java Servlet/JSP + MySQL)

The frontend is deliberately structured for a later backend:

1. **Single data choke point** — `js/services/store.js`. All reads/writes flow through it.
   Replace `localStorage` with `fetch()` and the whole app talks to your servlets.
2. **Async service functions** already return Promises with simulated latency, so wiring real
   endpoints is a drop-in (loading/error/empty states already exist in every page).
3. **Suggested endpoints:**
   ```
   POST /api/login
   GET  /api/activities?studentId=&status=&category=&search=
   POST /api/activities
   POST /api/verify            { activityId, action: approve|reject, remark }
   GET  /api/profile/{studentId}
   GET  /api/certificates/{certificateId}
   GET  /api/reports.csv?department=&year=&status=
   ```
4. **Security posture already modelled in the UI:**
   - passwords are stripped from every user object before rendering (`authService.sanitize`)
   - public portfolio exposes verified achievements + opted-in links only
   - student-added links are never shown with a verification badge
   - no real personal data in the demo dataset
5. **Schema sketch** for MySQL:
   `students`, `faculty`, `activities`, `activity_history`, `notifications`, `social_links`,
   `certificates` — mirroring the objects in `js/data/mockData.js`.

---

## 5. Browser support & notes

- Modern evergreen browsers (Chrome, Edge, Firefox, Safari). Uses ES2017 (async/await), CSS Grid, Flexbox.
- Charts are hand-rolled SVG — **zero external dependencies**, works offline.
- Data persists in `localStorage` (`campuscred.db.v1`). Reset from **Profile → Danger zone** or
  **Admin → Settings → Reset demo data**.
- Print styles let you "Download PDF" for the resume and reports via the browser's Save-as-PDF.
- The QR code is a deterministic visual stand-in (camera scanning is unavailable in a static demo);
  "Simulate scan" decodes a real certificate from the demo registry.

---

**Project Name:** CampusCred · **Problem Statement:** SKIT021
**Core Workflow:** Student Submission → Faculty Verification → Verified Achievement Record → Analytics, Portfolio & Resume
