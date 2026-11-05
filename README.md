# ARLATHI FITNESS — Gym Management System

A complete, production-ready gym management web application built with **React (Vite)**, **Firebase Authentication**, **Cloud Firestore**, **Firebase Storage**, **Tailwind CSS**, **Lucide React**, **Recharts**, **jsPDF** and **SheetJS (XLSX)**.

Everything is real: every form writes to Firestore, every dashboard number is computed from Firestore data, and every page is protected by Firebase Authentication and Firestore Security Rules.

---

## 1. Features

### Authentication
- Email/password login directly through Firebase Authentication.
- Only accounts that already exist in your Firebase project can sign in.
- Session persistence, protected routes, automatic redirect, logout, clear error messages.
- Automatic `users/{uid}` profile provisioning on first login (role `admin`).

### Dashboard
- 10 live statistic cards: members, active/inactive members, trainers, employees, expected payments, received payments, outstanding debts, monthly expenses, monthly revenue.
- Charts: 6-month income vs unpaid vs expenses, active vs inactive members, monthly income, member registrations, payment method breakdown.
- Recent payments, expiring memberships, top debts, live alerts and quick actions.

### Members
- Full CRUD with profile photo upload to Firebase Storage.
- Auto-generated unique Member ID (`ARL-XXXXXX`).
- Membership levels (Basic, Standard, Premium, VIP), status, start/expiry dates, monthly fee, trainer assignment, notes.
- Search, level/status filters, pagination, detail profile page.
- Per-member payment history and full debt history.
- Export to PDF/Excel, print directory, statement PDF.

### Trainers
- Full CRUD, profile photo, specialization, experience, salary, working days/hours, sessions per week, employment status.
- Trainer profile with assigned members, training timetable, attendance marking, salary payments (recorded as expenses) and payment history.

### Employees
- Full CRUD, photo, position, salary, employment date, working hours, status.
- Attendance marking, salary payments (recorded as expenses), performance notes and salary history.

### Training Schedule
- Daily, weekly and monthly calendar views.
- Multiple members per session, rooms, descriptions.
- Automatic overlapping-session prevention for the same trainer.
- Trainer filter and recurring weekly timetable.

### Payments
- Record monthly/partial payments with automatic balance calculation.
- Duplicate monthly charges are blocked at both UI and transaction level.
- Payment methods: Cash, EVC Plus, Zaad, Sahal, Bank Transfer, Other.
- Payment status: Paid, Partially Paid, Unpaid.
- Receipt modal, print, PDF download, payment search/filter/pagination.
- Daily, weekly and monthly collection totals.
- Void/correct a payment: debt balance is recalculated in a Firestore transaction and the action is written to `auditLogs`.

### Debts
- One billing record per member per period (`debts/{memberDocId}_{year}_{month}`).
- Generate monthly charges for all active members with duplicate protection.
- Automatic outstanding balance, partial payment tracking, old debt retention.
- Debt payments update `debts` and write a `payments` record without deleting history.
- Debt history per member, filters, search, PDF/Excel export.

### Expenses
- Categories: electricity, water, rent, equipment, employee/trainer salary, maintenance, cleaning, transport, other.
- CRUD, date/month/year filters, totals by day/month/year, category chart, PDF/Excel export.

### Financial Reports
- Monthly and annual reporting modes.
- Collected revenue, expected revenue, outstanding debt, expenses, employee salaries, trainer salaries, net income.
- Monthly summary table for the selected year, annual summary table, charts.
- `Net Income = Collected Revenue − Total Expenses` (unpaid fees are never counted as revenue).
- PDF/Excel export.

### Notifications
- Automatically generated from live data: expiring memberships, unpaid/partial fees, today's sessions and employee attendance issues.
- Read state persisted in the `notifications` collection, unread badge in sidebar/topbar.

### WhatsApp Communication
- Member WhatsApp fields: country code `+252` + 9-digit local number, verification flag and explicit opt-in.
- Individual messages with ready-made templates (welcome, expiry, payment reminder) and live preview.
- Bulk campaigns with level/status filters, opt-in/valid-number enforcement and per-recipient progress.
- Group tab for saving a group invite link, copying invitations and preparing announcements.
- Message history with status tracking (`prepared` → `opened` → `sent` → `delivered`), filters and Excel export.
- **Click-to-chat only:** messages open through `wa.me` links. The app cannot send messages automatically, create groups, or confirm delivery without a WhatsApp Business Cloud API backend.

### Bilingual Interface (English / Somali)
- Lightweight i18n layer in `src/i18n` with `en` and `so` dictionaries and `t(key, vars)` interpolation.
- Language switcher in the sidebar, topbar and login screen; preference is stored in `localStorage` and saved to `settings.preferredLanguage`.
- Data values (levels, statuses, payment methods, etc.) stay in English in Firestore and are translated only for display.

### UI/UX
- Premium black/dark-gray/white/orange/red fitness identity.
- Responsive sidebar, dark/light mode, loading states, toasts, confirmation dialogs, pagination, empty states.

---

## 2. Project structure

```
arlathi-fitness/
├── public/
│   ├── _redirects              # Netlify SPA routing
│   └── favicon.svg
├── src/
│   ├── components/             # Layout, forms, reusable UI
│   ├── context/                # Auth, Theme, Toast, Notifications, Language
│   ├── firebase/firebaseConfig.js
│   ├── hooks/useCollection.js
│   ├── i18n/                   # English + Somali dictionaries and translate()
│   ├── lib/
│   │   ├── billing.js          # Debt IDs, monthly charge generation, audit log
│   │   ├── constants.js
│   │   ├── exporters.js        # PDF / Excel / print helpers
│   │   ├── payments.js         # Payment + debt transactions, expenses
│   │   ├── storage.js          # Firebase Storage uploads
│   │   ├── whatsapp.js         # Number normalization, wa.me links, message logs
│   │   └── utils.js
│   ├── pages/                  # Login, Dashboard, Members, Trainers, Employees,
│   │                           # Schedule, Payments, Debts, Expenses, Reports,
│   │                           # Notifications, WhatsApp (+ tabs), Settings
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── firestore.rules
├── firestore.indexes.json
├── storage.rules
├── firebase.json
├── .firebaserc
├── netlify.toml
├── .env.example
└── package.json
```

---

## 3. Firebase setup

1. Create a project at <https://console.firebase.google.com>.
2. **Build → Authentication → Sign-in method**: enable **Email/Password**.
3. **Build → Authentication → Users**: add at least one administrator account (email + password). This is the account you will use to sign in — the app never creates fake accounts.
4. **Build → Firestore Database**: create a database (production mode is fine; rules are deployed below).
5. **Build → Storage**: enable storage for member/trainer/employee/logo photos.
6. **Project settings → General → Your apps**: create a Web app and copy the config values.
7. Copy `.env.example` to `.env` and fill in:

```env
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
```

8. Install the Firebase CLI, log in, and select the project:

```bash
npm install -g firebase-tools
firebase login
# edit .firebaserc or run:
firebase use YOUR_FIREBASE_PROJECT_ID
```

9. Deploy security rules and indexes:

```bash
firebase deploy --only firestore:rules,firestore:indexes,storage
```

> The rules allow any authenticated account to use the gym system. On first login the app creates `users/{uid}` with role `admin`. To restrict writes to specific people, remove their account from Firebase Authentication or change their `users/{uid}.role` to a non-admin value after changing the rules accordingly.

---

## 4. Run locally

```bash
npm install
npm run dev
```

Open the printed local URL and sign in with the Firebase Authentication account created in step 3.

Production build:

```bash
npm run build
npm run preview
```

---

## 5. Netlify deployment

1. Push the repository to GitHub/GitLab/Bitbucket.
2. In Netlify: **Add new site → Import an existing project**.
3. Build command: `npm run build` — Publish directory: `dist` (already set in `netlify.toml`).
4. Add environment variables under **Site configuration → Environment variables**:

```
VITE_FIREBASE_API_KEY
VITE_FIREBASE_AUTH_DOMAIN
VITE_FIREBASE_PROJECT_ID
VITE_FIREBASE_STORAGE_BUCKET
VITE_FIREBASE_MESSAGING_SENDER_ID
VITE_FIREBASE_APP_ID
```

5. Deploy. SPA redirects are provided by `netlify.toml` and `public/_redirects`.
6. In Firebase Console → **Authentication → Settings → Authorized domains**, add your Netlify domain (e.g. `your-site.netlify.app`).
7. Firestore/Storage rules are the same as above; no server credentials are used in the frontend.

Firebase Hosting is also supported:

```bash
npm run build
firebase deploy --only hosting
```

---

## 6. Firestore collections

| Collection | Purpose | Document ID |
| --- | --- | --- |
| `users` | Admin profile, role, last login | Firebase Auth UID |
| `members` | Gym members + membership data | auto |
| `trainers` | Gym instructors | auto |
| `employees` | Gym staff | auto |
| `trainingSchedules` | Recurring weekly sessions | auto |
| `payments` | Every payment transaction / receipt | auto |
| `debts` | One billing obligation per member per period | `{memberDocId}_{YYYY}_{MM}` |
| `expenses` | Gym running costs and salary payments | auto |
| `attendance` | Trainer/employee attendance | `trainer_{id}_{date}` / `employee_{id}_{date}` |
| `notifications` | Notification read state | deterministic alert ID |
| `whatsappMessages` | Prepared/opened/sent WhatsApp message log | auto |
| `whatsappCampaigns` | Bulk campaign metadata and progress | auto |
| `settings` | Gym profile, language, WhatsApp group | `main` |
| `auditLogs` | Sensitive action trail | auto |

Common fields: `createdAt`, `updatedAt`, `createdBy`/`createdByEmail` where applicable. All financial transactions store member references (`memberDocId`), billing period (`month`, `year`, `period`) and receipt numbers.

### Key financial rules
- A member + month + year can only have **one** debt record and payments are blocked once the period is fully paid.
- Partial payments add to `debts.amountPaid`, recalculate `debts.outstanding` and `debts.status` inside a Firestore transaction.
- Voiding a payment reverses the debt balance and preserves an audit log entry.
- Reports count only `payments.amountPaid` as collected revenue.
- Unpaid fees remain in `debts` and are reported as outstanding, never revenue.

---

## 7. Security rules summary

- Every read/write requires `request.auth != null`.
- `users/{uid}` can only be created by the matching authenticated UID and must use role `admin` or `staff`.
- Financial records (`payments`, `debts`, `expenses`, `settings`, `auditLogs`) require an authenticated administrator for modifications/deletions.
- Field/type validation is enforced for members, trainers, employees, schedules, payments, debts, expenses, attendance and audit logs.
- `notifications` can only be written with the caller's own `userId`.
- `whatsappMessages` and `whatsappCampaigns` require authentication, validate message type/status, and only administrators can delete logs.
- `auditLogs` are append-only.
- Storage writes only allow images under 5MB for authenticated users.
- Unmatched document paths are denied.

---

## 8. Verification checklist

1. Sign in with a Firebase Authentication account → dashboard loads.
2. Log out → protected routes redirect to `/login`.
3. Add/edit a member, upload a photo → appears in Firestore and Storage.
4. Add a trainer/employee → visible in lists, profiles and dashboard counts.
5. Create a schedule → overlapping session for the same trainer is rejected.
6. Generate monthly charges → one debt per active member/period.
7. Record a partial payment → payment receipt generated, debt becomes `Partially Paid`.
8. Pay the remaining balance → debt becomes `Paid`, further payments are blocked.
9. Export payments/members/debts/expenses/reports to PDF and Excel.
10. Add an expense → monthly expenses and net income update on reports.
11. Mark trainer/employee attendance → notification/attendance history updates.
12. Add a WhatsApp number to a member, verify it and enable opt-in → the member appears in eligible bulk recipients.
13. Prepare an individual WhatsApp message → WhatsApp opens, and the message appears in history with status `opened`.
14. Create a bulk campaign → one `whatsappMessages` record per eligible recipient is created.
15. Switch the interface to Somali → navigation, login and WhatsApp screens change language without altering Firestore values.
16. Verify Firestore rules by trying to access data while signed out (must be denied).
17. Deploy to Netlify with environment variables and confirm login + Storage uploads work.

---

## 9. Notes

- Firebase client config is supplied via Netlify environment variables; **no Admin SDK or service account credentials are included**.
- Charts and tables use live Firestore snapshots; no demo data is seeded by the application.
- Data is loaded with real-time snapshot listeners and filtered client-side for speed; composite indexes are provided for larger deployments.
- Currency formatting defaults to USD (`$`). Gym identity can be edited in **Settings**.
- WhatsApp integration is intentionally limited to `wa.me` click-to-chat. It does not automate sending, group creation, or delivery receipts; those require the WhatsApp Business Cloud API with a trusted backend.
- Somali translations live in `src/i18n/so.js`. Missing keys fall back to English, so new UI keys can be added safely.
