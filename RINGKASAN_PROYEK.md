# AAA Sport Academy — Ringkasan Proyek

Dokumen ini merangkum **seluruh hasil pembangunan aplikasi AAA Sport Academy**: halaman apa saja yang ada, cara kerja sistem, struktur database, fitur per modul, keputusan teknis penting, dan status terkini.

- **Nama aplikasi:** AAA Sport Academy
- **Jenis:** Website sekolah renang (landing page publik + dashboard manajemen privat)
- **Status:** Fungsional penuh dengan database (bukan mock), sudah melewati audit UI 14 issue
- **Terakhir diperbarui:** 6 November 2026

---

## Daftar Isi

1. [Ringkasan Eksekutif](#1-ringkasan-eksekutif)
2. [Tech Stack](#2-tech-stack)
3. [Struktur Direktori](#3-struktur-direktori)
4. [Persiapan & Cara Menjalankan](#4-persiapan--cara-menjalankan)
5. [Cara Kerja Sistem](#5-cara-kerja-sistem)
6. [Database](#6-database)
7. [Role & Hak Akses](#7-role--hak-akses)
8. [Halaman Publik](#8-halaman-publik)
9. [Halaman Dashboard per Modul](#9-halaman-dashboard-per-modul)
10. [API Endpoint](#10-api-endpoint)
11. [Fitur Lintas Modul](#11-fitur-lintas-modul)
12. [Jaminan Integritas Data](#12-jaminan-integritas-data)
13. [Design System & Perbaikan UI](#13-design-system--perbaikan-ui)
14. [Data Seed & Akun Demo](#14-data-seed--akun-demo)
15. [Validasi yang Sudah Dilakukan](#15-validasi-yang-sudah-dilakukan)
16. [Status & Pekerjaan Lanjutan](#16-status--pekerjaan-lanjutan)
17. [Dokumen Terkait](#17-dokumen-terkait)

---

## 1. Ringkasan Eksekutif

AAA Sport Academy adalah aplikasi web manajemen akademi renang dengan tiga program utama dan satu layanan privat:

| Kode DB | Nama tampilan | Karakter program |
| --- | --- | --- |
| `AAA` | AAA Sport Academy | Pengenalan air, teknik dasar, anak pemula |
| `A3` | A3 | Pengembangan teknik gaya, endurance, start/turn |
| `SWDC` | SWDC Swimmer | Latihan intensif dan persiapan kompetisi |
| `PRIVATE` | Private 1-on-1 | Slot privat kapasitas 1 siswa, **level mengikuti level siswa** |

Sistem terdiri dari dua sisi:

1. **Landing page publik** (`/`) — profil akademi, promo, program, testimoni, blog, FAQ, dan formulir pendaftaran yang mengalir menjadi **Lead** di CRM.
2. **Dashboard privat** (`/dashboard/*`) — manajemen siswa, pelatih, staff, tempat latihan, jadwal, booking privat, raport, keuangan, marketing, dan pengaturan sistem.

Prinsip utama yang dipegang di seluruh aplikasi:

- **Full database.** Tidak ada data bisnis mock, tidak ada mutasi yang hanya hidup di `localStorage`, tidak ada role switcher palsu. Settings sistem disimpan di tabel `system_settings`, bukan di browser.
- **Server-first.** Halaman membaca data langsung dari Prisma di Server Component, lalu menyerahkan data awal ke komponen client untuk interaksi.
- **Role-aware.** Setiap halaman, menu, dan endpoint difilter menurut role dan `programScope` di server — bukan hanya disembunyikan di UI.
- **Scope program.** ADMIN hanya melihat/mengelola program miliknya; SUPERADMIN dan OWNER bersifat global.

---

## 2. Tech Stack

### Runtime & Framework

| Kategori | Teknologi | Versi |
| --- | --- | --- |
| Framework | Next.js (App Router, JavaScript) | 16.4.3 |
| UI Library | React / React DOM | 19.3.0 |
| Bahasa | JavaScript ES6+ (tanpa TypeScript) | — |
| Styling | Tailwind CSS + `@tailwindcss/turbopack` | 4.3.3 |
| Animasi | `tw-animate-css` | 1.4.0 |
| Node.js | minimum | ≥ 20.9.0 |

### UI & Interaksi

| Kategori | Teknologi | Versi |
| --- | --- | --- |
| Ikon | `lucide-react` | 1.63.0 |
| Primitif UI | Radix UI (dialog, dropdown, popover, tabs, select, switch, tooltip, accordion, avatar, scroll-area, separator, label, checkbox, slot) | 1.3–1.4 |
| Variant helper | `class-variance-authority` | 0.7.1 |
| Utility class | `clsx` + `tailwind-merge` | 2.1 / 3.7 |
| Toast / notifikasi | `sonner` | 2.0.8 |
| Theme | `next-themes` | 0.4.6 |
| Chart | `recharts` | 3.10.1 |
| Shadcn CLI/schema | `shadcn` | 4.23.0 |

### Form, Validasi, Data

| Kategori | Teknologi | Versi |
| --- | --- | --- |
| State lokal | React Hooks (`useState`, `useMemo`, `useCallback`, `useEffect`, `useRef`) | — |
| Form | `react-hook-form` | 7.91.0 |
| Skema validasi | `zod` | 4.6.7 |
| Bridge form–zod | `@hookform/resolvers` | 6.0.2 |
| Server state (terpasang) | `@tanstack/react-query` | 5.105.0 |
| Tanggal | `date-fns` | 4.4.0 |

> **Catatan jujur soal TanStack Query:** paket sudah terpasang di `package.json`, tetapi implementasi data fetching saat ini memakai pola **Server Component + data awal (initial props) + client API module** (`lib/*/*Client.js`) dengan `fetch` langsung dan `toast` Sonner. Belum ada `QueryClientProvider` atau `useQuery` di kode. Ini bukan bug, hanya berarti TanStack Query masih menjadi opsi untuk migrasi berikutnya.

### Database & Keamanan

| Kategori | Teknologi | Versi |
| --- | --- | --- |
| ORM | Prisma (`prisma-client-js`) | 6.19.3 |
| Database | MySQL | — |
| Hash password | `bcryptjs` (cost 10) | 3.0.3 |
| Sanitasi HTML | `sanitize-html` (server only) | 2.17.5 |

---

## 3. Struktur Direktori

```
aaa-sport-academy/
├── app/
│   ├── layout.js                      # Root layout: font Inter+Manrope, Toaster Sonner
│   ├── globals.css                    # Tailwind v4, tema, CSS .mobile-card-table
│   ├── page.js                        # Landing page publik
│   ├── login/page.js                  # Halaman login
│   ├── blog/[slug]/page.js            # Detail artikel publik
│   ├── dashboard/                     # Area privat (auth guard di layout)
│   │   ├── layout.js                  # Guard + DashboardShell + Suspense
│   │   ├── page.js                    # Beranda / Ringkasan per role
│   │   ├── students/page.js           # Data Master Siswa
│   │   ├── data-master/{staff,pelatih,tempat-latihan,kolam}/page.js
│   │   ├── operasional/{page.js,jadwal,raport}/page.js
│   │   ├── schedule/[level]/page.js   # Kalender per program (aaa/a3/swdc/private)
│   │   ├── finance/page.js            # Transaksi / Langganan / Laporan
│   │   ├── my-package/page.js         # Paket Saya (self-service siswa)
│   │   ├── marketing/{layout.js,packages,campaign,content,leads}/page.js
│   │   └── settings/page.js           # Profil + konfigurasi sistem
│   └── api/                           # 44 Route Handler (daftar di §10)
├── components/
│   ├── ui/                            # StatCard, PageHeader, AdvancedFilters,
│   │                                  # RowActionButton, Table, Card, Button, Badge
│   ├── dashboard/                     # Shell, menu, global search, notifikasi
│   ├── data-master/                   # Student/Coach/Staff/TrainingVenue manager
│   ├── operational/                   # Kalender, jadwal keseluruhan, raport
│   ├── finance/                       # Finance sections, modal, Paket Saya
│   ├── marketing/                     # Katalog, campaign, content hub, leads
│   ├── settings/                      # Profil & sistem
│   ├── auth/                          # LoginForm + skeleton
│   └── landing/                       # Navbar, promo, testimoni, FAQ, CTA
├── hooks/                             # 14 custom hook (satu per fitur interaktif)
├── lib/
│   ├── auth/                          # Session, token, RBAC, rate limit, profil
│   ├── dashboard/                     # Overview, notifikasi, pencarian global
│   ├── data-master/                   # Service CRUD + client API
│   ├── operational/                   # Jadwal, occurrence, raport, date utils
│   ├── finance/                       # Transaksi, langganan, paket, renewal
│   ├── marketing/                     # Lead, promo, artikel, sanitasi HTML
│   ├── settings/                      # Settings, validasi, backup
│   ├── validations/                   # Skema Zod per domain
│   ├── api/routeHelpers.js            # ApiError, parse body, response helper
│   ├── db/prisma.js                   # Prisma singleton
│   ├── programs.js                    # Nama tampilan program
│   └── public-data.js                 # Query publik berbasis cache tag
├── prisma/
│   ├── schema.prisma                  # 25 model, 27 enum
│   └── seed.js                        # Seeder idempotent
├── public/images/hero-swimming.jpg
├── package.json / package-lock.json
└── *.md                               # Dokumen teknis per perbaikan
```

**Jumlah saat ini:** 25 model Prisma, 27 enum, 44 Route Handler, 14 custom hook, 46 file komponen.

---

## 4. Persiapan & Cara Menjalankan

### 4.1 Environment variables

Salin `.env.example` menjadi `.env`:

```env
# MySQL
DATABASE_URL="mysql://root:password@localhost:3306/aaa_sport_academy"

# Secret penandatangan cookie sesi (WAJIB diganti di production)
AUTH_SECRET="ganti-dengan-secret-acak-minimal-32-karakter"
```

Buat secret acak:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64url'))"
```

### 4.2 Install dependency

```bash
npm ci
```

> Gunakan `npm ci`, bukan `npm install`, agar versi mengikuti `package-lock.json` yang sudah tervalidasi.

### 4.3 Generate Prisma Client

```bash
npx prisma@6.19.3 generate
```

> Pin versi **6.19.3**. Prisma 7.x pernah gagal (`P1012`) pada format `datasource.url` di proyek ini.

### 4.4 Sinkronkan schema ke database

```bash
npm run db:push
```

Atau, bila memakai alur migration:

```bash
npm run db:migrate
```

### 4.5 Isi data seed

```bash
npm run seed
```

Seeder bersifat **idempotent**: menjalankan ulang akan menghasilkan data yang sama tanpa duplikasi (guard memakai `deleteMany` dan upsert pada kunci unik).

### 4.6 Jalankan aplikasi

```bash
npm run dev     # development
npm run build   # production build (prisma generate + next build)
npm start       # jalankan hasil build
```

### 4.7 Script yang tersedia

| Script | Fungsi |
| --- | --- |
| `npm run dev` | Dev server |
| `npm run build` | `prisma generate && next build` |
| `npm start` | Jalankan production server |
| `npm run lint` | ESLint |
| `npm run db:generate` | `prisma generate` |
| `npm run db:migrate` | `prisma migrate dev` |
| `npm run db:deploy` | `prisma migrate deploy` |
| `npm run db:push` | `prisma db push` |
| `npm run db:studio` | Prisma Studio |
| `npm run db:validate` | `prisma validate` |
| `npm run seed` | Jalankan seeder |

---

## 5. Cara Kerja Sistem

### 5.1 Alur request halaman

```
Request masuk
   ↓
app/dashboard/layout.js  (export const instant = false)
   ↓
<Suspense fallback={<DashboardShellSkeleton />}>
   ↓
requireCurrentUser()  → baca cookie → verifikasi token → ambil user dari DB
   ↓
getNewLeadCount(user)  → badge Leads awal (di-try/catch, tidak memblokir shell)
   ↓
<DashboardShell user>  → sidebar + topbar + children
   ↓
Halaman (Server Component) → requireFeatureAccess(feature) → query Prisma
   ↓
Data awal diteruskan ke Manager (Client Component) + hook
   ↓
Interaksi → client API module (fetch) → Route Handler → Service → Prisma
```

Keputusan desain yang penting:

- **Auth guard ada di layout, bukan middleware.** Karena itu seluruh `/dashboard/*` tidak mungkin diakses tanpa sesi valid.
- **`export const instant = false` pada layout dashboard.** Layout ini membaca cookie dan data user untuk gating, sehingga tidak punya static shell yang aman untuk diprerender. Konfigurasi ini hanya mengecualikan segment layout dashboard dari validasi Instant Navigation; halaman turunannya tetap divalidasi dan tetap memakai `<Suspense>` masing-masing.
- **`await connection()` dipertahankan** pada `getSession()`. Tanpa ini, `cookies()` dan `Date.now()` saat verifikasi token akan dianggap blocking prerender pada Cache Components.
- **Tidak memakai `runtime`/`dynamic`/`revalidate`/`fetchCache`** sebagai workaround — semuanya ditangani dengan Suspense boundary dan `instant`.

### 5.2 Pemisahan tampilan dan logika

Setiap fitur mengikuti pola tiga lapis:

| Lapis | Lokasi | Tanggung jawab |
| --- | --- | --- |
| Tampilan | `components/<domain>/*.jsx` | Markup, styling, aksesibilitas |
| Logika UI | `hooks/use*.js` | State, filter, form, pemanggilan API, toast |
| Data | `lib/<domain>/*Service.js` + `*Client.js` | Query Prisma (server) dan fetch (client) |

Contoh pemetaan:

| Halaman | Manager | Hook |
| --- | --- | --- |
| Data Siswa | `StudentManager.jsx` | `useStudentManager.js` |
| Data Pelatih | `CoachManager.jsx` | `useCoachManager.js` |
| Data Staff | `StaffManager.jsx` | `useStaffManager.js` |
| Tempat Latihan | `TrainingVenueManager.jsx` | `useTrainingVenueManager.js` |
| Finance | `FinanceManager.jsx` | `useFinanceManager.js` |
| Jadwal per program | `ScheduleCalendar.jsx` | `useScheduleCalendar.js` |
| Jadwal keseluruhan | `OverallScheduleManager.jsx` | `useOverallScheduleManager.js` |
| Raport | `StudentReportManager.jsx` | `useStudentReportManager.js` |
| Katalog Paket | `PackageCatalogManager.jsx` | `usePackageCatalogManager.js` |
| Paket Saya | `MyPackageManager.jsx` | `useMyPackageManager.js` |
| Login | `LoginForm.jsx` | `useLoginForm.js` |
| Shell dashboard | `DashboardShell.jsx` | `useDashboardShell.js` |
| Pencarian global | `GlobalSearch.jsx` | `useGlobalSearch.js` |
| Notifikasi | `NotificationMenu.jsx` | `useDashboardNotifications.js` |

### 5.3 Autentikasi & sesi

1. **Login** — `POST /api/auth/login` memvalidasi dengan Zod, lalu:
   - `checkLoginRateLimit()` memeriksa **dua bucket** di tabel `login_rate_limits` (HMAC-SHA256, IP dan username mentah tidak disimpan):
     - `IP + username`: maksimal **5 percobaan / 15 menit**
     - `IP`: maksimal **20 percobaan / 15 menit**
     - Blokir: **15 menit**
   - `authenticateUser()` mencari user berdasarkan **username**, lalu `bcrypt.compare`.
   - Bila username tidak ada, jalur tetap menjalankan `bcrypt.compare` terhadap **dummy hash** agar waktu respons tidak membocorkan keberadaan akun.
   - Sukses → `clearSuccessfulLoginAttempt()` + `setSessionCookie()`.
2. **Cookie sesi** — nama `aaa_session`, berisi token HMAC bertanda tangan (`AUTH_SECRET`), TTL **12 jam**, flag `httpOnly`, `sameSite=lax`, `secure` di production, `path=/`.
3. **Verifikasi tiap request** — `getCurrentUser()` memverifikasi tanda tangan, masa berlaku, lalu **membaca ulang user dari database** dan membandingkan `tokenVersion`. Token dengan versi berbeda langsung ditolak.
4. **Ganti password** — server menaikkan `User.tokenVersion`, sehingga **semua token lama otomatis tidak berlaku** di seluruh perangkat, lalu menerbitkan cookie baru untuk sesi yang sedang aktif.
5. **Logout** — `POST /api/auth/logout` menghapus cookie sesi.
6. **Redirect aman** — `getSafeRedirectPath()` mencegah open redirect pada parameter `?next=`.

> Kredensial login utama adalah **`username`**. `email` tetap unik, tetapi berfungsi sebagai data profil/kontak, bukan kredensial.

### 5.4 Otorisasi berlapis

| Lapis | Implementasi |
| --- | --- |
| Menu sidebar | `filterMenuByRole()` menyembunyikan item tanpa hak akses |
| Halaman (server) | `requireFeatureAccess(feature, nextPath)` → redirect bila tidak berhak |
| Layout segment | `app/dashboard/marketing/layout.js` memakai guard khusus Marketing |
| Route Handler | `requireApiUser([roles])` |
| Service (data) | Scope `programScope` diterapkan di query, misalnya `assertFinanceProgramAccess()` dan filter `level` |
| Kepemilikan baris | Contoh: COACH hanya boleh mengubah raport yang `coachId`-nya miliknya; STUDENT hanya melihat datanya sendiri |

### 5.5 Cache & revalidasi

- Landing page dan query publik memakai `'use cache'` + `cacheLife('hours')`, dengan tag:
  `promotions`, `articles`, `article:<slug>`, `program-content`, `packages`, `testimonials`, `faq-items`, `system-settings`, `training-venues`, `public-stats`, `public-landing`.
- Mutasi Marketing dan Settings memanggil `revalidateTag(tag, 'max')` dari Route Handler sehingga landing page ikut segar tanpa full rebuild.
- Tahun berjalan di footer memakai `'use cache'` + `cacheLife('max')`.
- Halaman dashboard **tidak** di-cache; semuanya request-time.

---

## 6. Database

Provider: **MySQL**. Nama tabel memakai `snake_case` lewat `@@map`, kolom multi-kata memakai `@map`.

### 6.1 Model (25)

| Kelompok | Model | Isi penting |
| --- | --- | --- |
| **Akun** | `User` | `username` unik (login), `email` unik (profil), `password` bcrypt, `role`, `programScope`, `specialization`, `isActive`, `tokenVersion` |
| | `Staff` | Staf non-pengajar; opsional terhubung ke `User`; `position`, `department`, `status` |
| **Akademi** | `Student` | `userId` opsional, `name`, `dateOfBirth`, `age`, `parentContact`, `healthHistory` (sensitif), `level`, `status` |
| | `Pool` | Tempat latihan: `name`, `location`, `depth`, `capacity`, `status`, `imageUrl` |
| **Jadwal** | `Schedule` | Jadwal berulang: `program`, `dayOfWeek`, `startTime`, `endTime`, `capacity`, relasi `trainingVenueId`, snapshot `poolName`, `lane`, `status`, periode efektif |
| | `Slot` | Kemunculan nyata per tanggal: `startsAt`, `endsAt`, snapshot `capacity`, `bookedCount`, `status`; unik `(scheduleId, startsAt)` |
| | `CoachAssignment` | Penugasan pelatih ke jadwal (berulang) atau slot (override); `role` LEAD/ASSISTANT/SUBSTITUTE |
| | `Booking` | Booking siswa pada slot; `status` CONFIRMED/CANCELLED/COMPLETED/NO_SHOW |
| **Raport** | `StudentReport` | `studentId`, `coachId`, snapshot `level`, `periodLabel`, `sessionsAttended`, `totalSessions`, `recommendation`, `status`, `publishedAt`; unik `(studentId, periodLabel)` |
| | `StudentReportItem` | Butir kompetensi: `competency`, `score` 1–5, `note`, `sortOrder` |
| **Keuangan** | `Package` | Katalog paket: `program`, `price`, `durationLabel`, `durationDays`, `totalSessions`, tempat latihan default, `isActive` |
| | `Subscription` | Langganan siswa: snapshot `packageName`/`program`/`price`, `startsAt`, `expiresAt`, `totalSessions`, `remainingSessions`, `status`, `autoRenew` |
| | `Transaction` | Invoice: `invoiceNumber` unik, `type`, `amount`, `paymentMethod`, `status`, `paidAt`, snapshot `program`/`poolName`, `referenceNumber` |
| **Marketing** | `Lead` | Prospek: data siswa & orang tua, `program`, `source`, `status`, `priority`, `assignedToId`, `nextFollowUpAt`, `trialScheduledAt`, `convertedStudentId` |
| | `LeadActivity` | Riwayat aktivitas lead: `type` NOTE/STATUS/ASSIGNMENT/SYSTEM, `title`, `description`, `actorId` |
| | `Promotion` | Promo/voucher/referral: `type`, `code` unik, `batchId`, `discountType`, `discountValue`, periode, kuota, `redeemedCount`, `status` |
| | `Article` | Blog: `slug` unik, `content` LongText (HTML tersanitasi), `category`, `tags`, gambar utama, SEO, `status`, `publishedAt`, `viewCount` |
| **Sistem** | `SystemSetting` | Singleton `id = "default"`; kolom JSON `general`, `financial`, `notification`, `siteContent`; `updatedById` |
| | `NotificationTemplate` | Template WhatsApp/email: `channel`, `eventKey`, `subject`, `message`, `isActive` |
| | `DatabaseBackup` | Snapshot JSON: `fileName` unik, `format`, `sizeBytes`, `content` LongText |
| | `NotificationRead` | Tanda baca notifikasi per user; unik `(userId, notificationKey)` |
| | `LoginRateLimit` | Bucket rate limit login; `keyHash` unik (HMAC), `attempts`, `blockedUntil`, `expiresAt` |
| **Konten publik** | `ProgramContent` | Konten per program (`program` unik): tagline, deskripsi, audiens, label jadwal, fitur |
| | `Testimonial` | Testimoni landing: `name`, `roleLabel`, `quote`, `rating`, `sortOrder` |
| | `FaqItem` | FAQ landing: `question` unik, `answer`, `sortOrder` |

### 6.2 Enum (27)

| Enum | Nilai |
| --- | --- |
| `UserRole` | `SUPERADMIN`, `OWNER`, `ADMIN`, `COACH`, `STUDENT` |
| `PoolStatus` | `ACTIVE`, `MAINTENANCE` |
| `StaffStatus` | `ACTIVE`, `ON_LEAVE`, `INACTIVE` |
| `StudentLevel` | `AAA`, `A3`, `SWDC` |
| `StudentStatus` | `ACTIVE`, `PENDING`, `COMPLETED`, `INACTIVE` |
| `ScheduleProgram` | `AAA`, `A3`, `SWDC`, `PRIVATE` |
| `DayOfWeek` | `MONDAY` … `SUNDAY` |
| `ScheduleStatus` | `ACTIVE`, `INACTIVE`, `CANCELLED` |
| `SlotStatus` | `OPEN`, `FULL`, `CLOSED`, `CANCELLED` |
| `CoachAssignmentRole` | `LEAD`, `ASSISTANT`, `SUBSTITUTE` |
| `CoachAssignmentStatus` | `ASSIGNED`, `CONFIRMED`, `COMPLETED`, `CANCELLED` |
| `BookingStatus` | `CONFIRMED`, `CANCELLED`, `COMPLETED`, `NO_SHOW` |
| `ReportStatus` | `DRAFT`, `PUBLISHED` |
| `ReportRecommendation` | `STAY`, `LEVEL_UP`, `EXTRA_PRACTICE` |
| `PaymentMethod` | `CASH`, `TRANSFER`, `QRIS` |
| `TransactionType` | `SUBSCRIPTION_PAYMENT`, `PRIVATE_VOUCHER`, `REGISTRATION`, `OTHER` |
| `TransactionStatus` | `PAID`, `PENDING`, `FAILED`, `CANCELLED`, `REFUNDED` |
| `SubscriptionStatus` | `ACTIVE`, `PENDING`, `EXPIRED`, `CANCELLED`, `COMPLETED` |
| `PromotionType` | `FLASH_SALE`, `REFERRAL`, `VOUCHER`, `SEASONAL`, `PARTNER` |
| `DiscountType` | `PERCENTAGE`, `FIXED_AMOUNT`, `FREE_SESSION` |
| `PromotionStatus` | `DRAFT`, `SCHEDULED`, `ACTIVE`, `INACTIVE`, `EXPIRED`, `REDEEMED` |
| `ArticleStatus` | `DRAFT`, `SCHEDULED`, `PUBLISHED`, `ARCHIVED` |
| `LeadSource` | `WEBSITE`, `LANDING_PAGE`, `INSTAGRAM`, `WHATSAPP`, `REFERRAL`, `WALK_IN`, `EVENT`, `OTHER` |
| `LeadStatus` | `NEW`, `CONTACTED`, `WAITING_RESPONSE`, `TRIAL_SCHEDULED`, `TRIAL_COMPLETED`, `REGISTERED`, `NOT_INTERESTED`, `UNREACHABLE` |
| `LeadPriority` | `LOW`, `MEDIUM`, `HIGH` |
| `LeadActivityType` | `NOTE`, `STATUS`, `ASSIGNMENT`, `SYSTEM` |
| `NotificationChannel` | `WHATSAPP`, `EMAIL` |

### 6.3 Relasi inti

```
User 1─0..1 Student        (akun login siswa)
User 1─0..1 Staff          (akun login staff)
User 1─* CoachAssignment   (pelatih yang ditugaskan)
User 1─* StudentReport     (pelatih penilai)
User 1─* Lead              (lead yang di-assign)
User 1─* LeadActivity      (aktor aktivitas)
User 1─* NotificationRead
User 1─* DatabaseBackup    (pembuat backup)

Pool 1─* Schedule          (tempat latihan)
Pool 1─* Package           (tempat latihan default paket)

Schedule 1─* Slot          (kemunculan per tanggal)
Schedule 1─* CoachAssignment
Slot     1─* Booking
Slot     1─* CoachAssignment (override)

Student 1─* Subscription
Student 1─* Transaction
Student 1─* Booking
Student 1─* StudentReport
Student 1─0..1 Lead       (hasil konversi lead)

Package 1─* Subscription
Package 1─* Transaction

Subscription 1─* Transaction (pembayaran langganan)

StudentReport 1─* StudentReportItem
```

### 6.4 Strategi snapshot data

Beberapa nilai sengaja **diduplikasi** ke tabel turunan agar histori tidak berubah saat master diperbarui:

| Tabel | Snapshot | Alasan |
| --- | --- | --- |
| `Schedule` | `poolName` | Nama kolam tetap terbaca walau master berubah |
| `Slot` | `capacity` | Kapasitas historis tidak ikut berubah |
| `Subscription` | `packageName`, `program`, `price` | Harga saat pembelian tidak berubah |
| `Transaction` | `program`, `poolName` | Laporan historis tetap konsisten |
| `StudentReport` | `level` | Level siswa saat raport dibuat |

### 6.5 Indeks

Index dibuat pada kolom yang memang menjadi jalur filter/urut utama: `role`, `programScope`, `isActive`, `level`, `status`, `program`, `dayOfWeek`, `startsAt`, `paidAt`, `expiresAt`, `nextFollowUpAt`, `publishedAt`, `createdAt`, dan beberapa composite seperti `(level, status)`, `(status, paidAt)`, `(status, expiresAt)`, `(status, nextFollowUpAt)`, `(scheduleId, startsAt)`.

### 6.6 Catatan penerapan schema

- Folder `prisma/migrations` belum ada. Perubahan schema diterapkan dengan `npm run db:push`.
- Penambahan terakhir berupa kolom `users.token_version` (`User.tokenVersion`) **harus diterapkan ke database user** melalui `db:push` atau migration sebelum login di lingkungan tersebut berjalan dengan benar.

---

## 7. Role & Hak Akses

Hierarki: **SUPERADMIN → OWNER → ADMIN → COACH → STUDENT**.

Sumber tunggal kebenaran: `lib/auth/permissions.js` (`FEATURE_ROLES`, `canAccessFeature`, `getProgramScope`, `canManageProgram`, `canPermanentlyDelete`).

| Role | Ringkasan wewenang |
| --- | --- |
| **SUPERADMIN** | God Mode. Semua fitur termasuk Settings sistem, backup, dan seluruh konfigurasi. |
| **OWNER** | Operasional bisnis penuh termasuk hapus permanen; **tidak** boleh mengubah Settings sistem. |
| **ADMIN** | Admin satu program (`programScope`). Kelola siswa, operasional, dan finance pada levelnya. **Tidak** mengelola Data Master global/Marketing/katalog paket, tidak hapus permanen, tidak membuka Settings sistem. |
| **COACH** | Lihat jadwal, daftar siswa, dan mengelola raport yang ia nilai sendiri. |
| **STUDENT** | Booking Private, lihat jadwal & booking sendiri, raport terbit sendiri, Paket Saya, Settings Profil. |

### Matriks fitur

| Feature key | SUPERADMIN | OWNER | ADMIN | COACH | STUDENT |
| --- | :-: | :-: | :-: | :-: | :-: |
| `dashboard` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `dataMasterStaff` | ✅ | ✅ | — | — | — |
| `dataMasterCoaches` | ✅ | ✅ | — | — | — |
| `dataMasterStudents` | ✅ | ✅ | ✅ | ✅ (baca) | — |
| `dataMasterVenues` | ✅ | ✅ | — | — | — |
| `operationalOverview` | ✅ | ✅ | ✅ | ✅ | — |
| `scheduleLevels` | ✅ | ✅ | ✅ (scope) | ✅ | ✅ |
| `studentReports` | ✅ | ✅ | ✅ (scope) | ✅ (miliknya) | ✅ (terbit saja) |
| `finance` | ✅ | ✅ | ✅ (scope) | — | — |
| `myPackage` | ✅ | — | — | — | ✅ |
| `marketing` | ✅ | ✅ | — | — | — |
| `settingsProfile` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `settingsSystem` | ✅ | — | — | — | — |

### Penerapan scope program

- `getProgramScope(actor)` hanya mengembalikan nilai bila role = `ADMIN` dan `programScope` termasuk `AAA`/`A3`/`SWDC`.
- `canManageProgram(actor, program, studentLevel)`:
  - SUPERADMIN/OWNER → selalu `true`.
  - ADMIN → `program === scope`; untuk `PRIVATE` dicek `studentLevel === scope`.
- Filter scope diterapkan di query service (mis. `scopedTransactionWhere`, `scopedSubscriptionWhere`, filter `Student.level`), bukan hanya di UI.
- Menu program di sidebar ikut disaring: ADMIN hanya melihat program miliknya (Private tetap tampil).

---

## 8. Halaman Publik

### 8.1 Landing page — `/`

| Urutan | Bagian | `id` | Isi |
| --- | --- | --- | --- |
| 1 | Hero | `#beranda` | Headline, CTA "Daftar Sekarang" & "Lihat Program", 3 statistik ringkas dari database, gambar hero, dua kartu mengambang (Belajar Aman, Kelompok Kecil) |
| 2 | Fitur akademi | — | Grid kartu fitur dari `SystemSetting.siteContent` (`academyFeatures`) dengan ikon dinamis |
| 3 | Promo | `#promo` | Promo aktif beserta kode yang bisa disalin (`CopyPromoButton`) |
| 4 | Program | `#program` | Kartu per program dari `ProgramContent` + paket aktif |
| 5 | Testimoni | `#testimoni` | Slider testimoni dari tabel `Testimonial` |
| 6 | Blog | `#blog` | 3 artikel terbit terbaru |
| 7 | FAQ | `#faq` | Accordion dari tabel `FaqItem` |
| 8 | Pendaftaran | `#daftar` | Formulir lead + tombol WhatsApp |

Detail teknis landing page:

- Skip link "Lewati ke konten" untuk aksesibilitas keyboard.
- Semua data berasal dari database (`lib/public-data.js`) dengan cache tag dan revalidasi.
- Nama bisnis, tagline, logo, jam operasional, dan nomor telepon mengikuti Settings sistem — bukan hardcoded.
- Nomor telepon dinormalisasi ke format `wa.me`.
- Nomor tahun di footer memakai `'use cache'` + `cacheLife('max')` agar tidak memicu blocking prerender.

### 8.2 Formulir pendaftaran (Lead capture)

- Komponen: `components/landing/RegistrationCta.jsx`.
- Field: nama siswa, nama orang tua, nomor WhatsApp, program, preferensi tempat latihan, dan **honeypot** `website`.
- Submit → `POST /api/leads` → `createPublicLead()`.
- Endpoint publik **tidak** mengembalikan PII; hanya `{ accepted: true }`.
- Duplikasi nomor telepon dideteksi dan dijawab `200` agar tidak membocorkan status internal.
- Setelah sukses, tombol WhatsApp chat muncul dengan pesan yang sudah terisi.

### 8.3 Detail artikel — `/blog/[slug]`

- `generateMetadata()` memakai `seoTitle`/`seoDescription` bila ada.
- HTML artikel **disanitasi di server** dengan `sanitizeArticleHtml()` sebelum dirender.
- Artikel terkait ditampilkan berdasarkan kategori.
- Artikel yang tampil publik adalah `PUBLISHED` yang `publishedAt`-nya sudah lewat, atau `SCHEDULED` yang `scheduledAt`-nya sudah lewat. Selain itu `notFound()`.

### 8.4 Login — `/login`

- Form: username + password, toggle tampilkan password, pesan error per field.
- Mendukung `?next=` yang divalidasi `getSafeRedirectPath()`.
- Sudah login → langsung redirect ke dashboard.
- Skeleton (`LoginFormSkeleton`) tampil selama proses request-time.

---

## 9. Halaman Dashboard per Modul

### 9.1 Beranda / Ringkasan — `/dashboard`

Data berasal dari `getDashboardOverview(user)` yang bercabang per role:

| Role | Isi |
| --- | --- |
| **Business** (SUPERADMIN/OWNER/ADMIN) | Siswa aktif, siswa pending, pendapatan bulan ini, pendapatan hari ini, agregat pembayaran pending, langganan segera berakhir, lead baru, sebaran siswa per level, tren transaksi 7 hari, daftar langganan segera berakhir, transaksi terbaru, jadwal hari ini, siswa terbaru |
| **COACH** | Jadwal mengajar hari ini, jumlah penugasan, jumlah draf raport, daftar draf raport, slot yang bisa diambil |
| **STUDENT** | Profil siswa, langganan aktif, booking mendatang, jadwal level hari ini, raport terbit terbaru, tagihan pending |

Prinsip:

- Tidak ada mock dan tidak ada role switcher. Isi mengikuti sesi dan agregat database.
- Sapaan beranda memakai `<h2>` (bukan `<h1>`) karena `<h1>` sudah dipakai judul halaman di topbar.
- KPI memakai komponen kanonis `StatCard`.

### 9.2 Data Master — Siswa, Pelatih, Staff, Tempat Latihan

| Halaman | Route | Siapa yang bisa mengelola |
| --- | --- | --- |
| Siswa | `/dashboard/students` | SUPERADMIN, OWNER, ADMIN (scope); COACH hanya baca |
| Pelatih | `/dashboard/data-master/pelatih` | SUPERADMIN, OWNER |
| Staff | `/dashboard/data-master/staff` | SUPERADMIN, OWNER |
| Tempat Latihan | `/dashboard/data-master/tempat-latihan` | SUPERADMIN, OWNER |
| Kolam (redirect) | `/dashboard/data-master/kolam` | diteruskan ke Tempat Latihan |

Fitur Data Siswa:

- Tabel siswa dengan pencarian, filter level, filter status.
- CRUD siswa; usia terisi otomatis dari tanggal lahir.
- `healthHistory` ditampilkan terbatas sesuai role.
- Status default siswa baru: **`PENDING`**.
- ADMIN terkunci pada level program miliknya saat membuat siswa.
- Hapus permanen hanya SUPERADMIN/OWNER; ADMIN cukup menonaktifkan lewat status.

Fitur Pelatih:

- CRUD akun COACH (nama, username, email, telepon, spesialisasi, status aktif).
- Spesialisasi program; email/username tidak diekspos ke client STUDENT.

Fitur Staff:

- CRUD data staff non-pengajar beserta status (`ACTIVE`, `ON_LEAVE`, `INACTIVE`).

Fitur Tempat Latihan:

- CRUD `Pool`: nama, lokasi, kedalaman, kapasitas, status, gambar.
- Dipakai sebagai referensi jadwal dan tempat latihan default paket.

### 9.3 Operasional

#### Ringkasan Operasional — `/dashboard/operasional`
Redirect ke Jadwal Keseluruhan.

#### Jadwal Keseluruhan — `/dashboard/operasional/jadwal`

- Dua tampilan: **Board mingguan per hari** (desktop `lg+`) dan **daftar tabel** (semua ukuran; menjadi kartu di mobile).
- Toolbar: pencarian + **Program** + **Hari**; sisanya (Tempat, Pelatih, Status) di **Filter lainnya (n)** dengan badge jumlah filter aktif dan Reset.
- Aksi baris: Ubah, Hapus (berlabel teks, bukan ikon-tooltip).
- Statistik: total jadwal, jadwal aktif, pelatih bertugas, dan jumlah jadwal yang belum punya pelatih (amber).

#### Kalender per Program — `/dashboard/schedule/[level]`

Level yang didukung: `aaa`, `a3`, `swdc`, `private`.

- **Desktop (`lg+`):** grid kalender 7 kolom (Senin–Minggu).
- **Mobile/tablet:** agenda vertikal per hari dengan kartu yang sama — tanpa paksaan scroll horizontal.
- Navigasi minggu (minggu dimulai **Senin**).
- Setiap kartu slot menampilkan program, jam, pelatih, jumlah terisi/sisa kapasitas, dan status slot.
- Manager: assign / ubah pelatih, buka/tutup slot.
- STUDENT (dan manajemen): tombol **Booking Slot** untuk slot Private yang kosong; booking milik sendiri bisa dibatalkan.
- Menampilkan **satu** notice kontekstual netral (bukan tumpukan banner amber).
- Level tidak valid dirender sebagai halaman error `<h2>` dengan tautan kembali.

#### Raport Siswa — `/dashboard/operasional/raport`

- CRUD raport per siswa & periode; satu siswa hanya punya satu raport per `periodLabel`.
- Katalog kompetensi default per level (AAA/A3/SWDC) dapat ditambah/diedit manual.
- Skor 1–5 dengan label: Perlu Bimbingan → Mulai Berkembang → Berkembang → Mahir → Sangat Mahir.
- Kehadiran manual (`sessionsAttended`/`totalSessions`) dan persentase kehadiran.
- Rekomendasi: Tetap di Level Ini / Naik ke Level Berikutnya / Perlu Latihan Tambahan.
- **Aturan penting:**
  - `DRAFT` tidak pernah tampil ke STUDENT (filter `status: PUBLISHED` di query untuk aktor siswa).
  - COACH hanya dapat mengubah raport dengan `coachId` miliknya; raport tanpa penilai harus ditetapkan lebih dulu oleh manajemen.
  - Menerbitkan raport dengan rekomendasi `LEVEL_UP` menaikkan level siswa mengikuti `NEXT_STUDENT_LEVEL` (`AAA → A3 → SWDC`).
- Aksi baris berlabel: Detail, Cetak, Terbitkan/Ke Draf, Ubah, Hapus.
- **Cetak** membuka dokumen HTML rapi di tab baru. Implementasinya membuka tab kosong lebih dulu, memutus `opener`, lalu menavigasi ke blob URL — sehingga pesan "popup diblokir" hanya muncul bila popup benar-benar diblokir.

### 9.4 Finance — `/dashboard/finance`

Grup Finance hanya berisi **tiga tab**:

| Tab | Query | Isi |
| --- | --- | --- |
| **Transaksi** | `?tab=transactions` | Invoice, status, metode bayar, konfirmasi lunas, kwitansi |
| **Langganan** | `?tab=subscriptions` | Masa aktif, sisa sesi, perpanjangan, auto-renew |
| **Laporan** | `?tab=reports` | Rekap pendapatan, per program, per tempat latihan, export CSV |

Penting: **Katalog Paket tidak ada di Finance.** Katalog dikelola di Marketing dan Finance membacanya **read-only** sebagai opsi transaksi/langganan dan dimensi laporan.

Fitur Transaksi:

- Toolbar: pencarian + **Periode** + **Status**; metode, program, dan tempat latihan masuk **Filter lainnya (n)**.
- Membuat transaksi: pilih siswa, paket (opsional), jenis, nominal, metode, tanggal, catatan.
- Nomor invoice berpola `INV-YYYYMMDD-XXXX` dan unik.
- Konfirmasi pembayaran `PENDING → PAID`:
  - mengisi `paidAt` mengikuti tanggal pembayaran,
  - memperpanjang/membuat langganan sesuai paket,
  - menaikkan status siswa `PENDING → ACTIVE`.
- Cetak kwitansi, kirim pengingat WhatsApp (template dari Settings), export CSV.
- Statistik: pendapatan hari ini, transaksi hari ini, menunggu pembayaran (amber), dan langganan aktif.

Fitur Langganan:

- Status: ACTIVE, PENDING, EXPIRED, CANCELLED, COMPLETED.
- Menampilkan sisa sesi (`remainingSessions/totalSessions`) dan masa berlaku.
- Perpanjangan memperpanjang `expiresAt` dan menambah sisa sesi.

Fitur Laporan:

- Periode laporan dapat dipilih; ringkasan berisi total pendapatan, jumlah transaksi lunas, rata-rata transaksi, jumlah pembayaran menunggu beserta nominalnya, dan total refund.
- Grafik tren harian serta rincian pengelompokan per **tempat latihan**, **program**, **metode pembayaran**, dan **paket**.
- Export CSV dengan proteksi formula injection (lihat §12).

### 9.5 Paket Saya — `/dashboard/my-package`

Halaman self-service untuk STUDENT (SUPERADMIN dapat membukanya dalam mode God Mode dengan data profilnya sendiri).

- Menampilkan profil siswa, langganan aktif, sisa sesi, dan riwayat transaksi.
- Ajukan perpanjangan: pilih paket aktif + metode pembayaran → membuat transaksi `PENDING`.
- Validasi server:
  - profil siswa harus terhubung,
  - siswa `INACTIVE` ditolak,
  - paket harus aktif,
  - **hanya boleh ada satu pengajuan `PENDING`** pada satu waktu (mencegah tagihan ganda).
- Halaman mengambil data lewat query yang sudah difilter per siswa (`getSubscriptionsByStudentId`, `getTransactionsByStudentId`) — tidak memuat seluruh tabel lalu menyaring di JavaScript.

### 9.6 Marketing — `/dashboard/marketing/*`

Akses: **SUPERADMIN dan OWNER** (guard di `app/dashboard/marketing/layout.js`).

#### Katalog Paket — `/dashboard/marketing/packages`

- Pemilik tunggal CRUD katalog: membuat, mengubah, mengaktifkan/nonaktifkan, menghapus paket.
- Field: nama, deskripsi, program, harga, label & durasi hari, total sesi (opsional), tempat latihan default.
- Statistik: paket aktif, paket nonaktif, langganan terkait, transaksi terkait.
- Aksi baris berlabel (Ubah/Hapus).
- Setiap mutasi memanggil `revalidateTag` untuk `packages`, `program-content`, dan `public-landing`.

#### Campaign Manager — `/dashboard/marketing/campaign`

- CRUD promo: tipe (flash sale, referral, voucher, seasonal, partner), diskon (persen/nominal/gratis sesi), periode, kuota, syarat minimum, batas maksimum diskon, program/tempat latihan.
- **Voucher Generator:** membuat banyak kode unik satu kali pakai dalam satu batch (`batchId`).
- **Referral Code Generator:** membuat kode referral personal.
- Kode dapat disalin dengan sekali klik.
- Statistik: campaign aktif, terjadwal, total redemption, voucher tersedia.
- Tabel memakai komponen `Table` bersama yang responsif.

#### Content Hub — `/dashboard/marketing/content`

- WYSIWYG editor artikel dengan toolbar; mendukung gambar unggulan lewat URL atau unggah file (disimpan ke database).
- Pengaturan publikasi: status (`DRAFT` / `SCHEDULED` / `PUBLISHED` / `ARCHIVED`) dan jadwal terbit.
- SEO: judul & deskripsi.
- **Sanitasi wajib di server**, termasuk untuk preview dan paste/drop.
- Statistik: published, scheduled, draft, total views.
- Aksi baris: Duplikat, Hapus (berlabel).

#### Leads / Funnel — `/dashboard/marketing/leads`

- Pipeline lead dengan status lengkap (NEW → CONTACTED → WAITING_RESPONSE → TRIAL_SCHEDULED → TRIAL_COMPLETED → REGISTERED / NOT_INTERESTED / UNREACHABLE).
- Filter status & sumber, pencarian, prioritas, penugasan ke user.
- Detail lead: riwayat aktivitas (`LeadActivity`), catatan internal, jadwal follow-up, jadwal trial.
- Statistik: lead baru, perlu follow-up (merah), trial dijadwalkan, persentase konversi.
- Badge jumlah lead baru muncul di sidebar (dan bersifat dinamis, bukan hardcoded).

### 9.7 Settings — `/dashboard/settings`

Satu route dengan lima tab (`?tab=`):

| Tab | Akses | Isi |
| --- | --- | --- |
| **Profil & Keamanan** | Semua role | Ubah nama, telepon, avatar, dan password sendiri |
| **General** | SUPERADMIN | Nama bisnis, nama legal, tagline, email, telepon, website, alamat, timezone, logo, jam operasional 7 hari |
| **Financial Rules** | SUPERADMIN | Pajak, denda keterlambatan, kebijakan refund, contoh perhitungan |
| **Notification Templates** | SUPERADMIN | Template WhatsApp/email per event, penanda token, preview |
| **Backup Database** | SUPERADMIN | Buat snapshot JSON, daftar histori, unduh file |

Perilaku UI yang sudah diperbaiki:

- Judul halaman tunggal (`<h1>` di topbar), header konten tanpa judul ganda.
- Tab aktif ditentukan oleh query string, sehingga judul dan highlight sidebar akurat.
- **Save bar** hanya muncul bila: role SUPERADMIN **dan** tab bukan Profil **dan** ada perubahan belum disimpan. Tidak ada lagi tombol simpan ganda atau offset `fixed` hardcoded.
- Peringatan amber hanya dipakai untuk peringatan nyata.

Catatan penting: Financial Rules saat ini **tersimpan sebagai konfigurasi**, belum menjadi mesin aturan yang otomatis menghitung pajak/denda pada invoice. Lihat §16.

---

## 10. API Endpoint

44 Route Handler di `app/api`. Semua endpoint privat memakai `requireApiUser([roles])`; endpoint publik disebut eksplisit.

### Auth

| Method | Path | Akses | Fungsi |
| --- | --- | --- | --- |
| POST | `/api/auth/login` | Publik | Login (rate limited) |
| POST | `/api/auth/logout` | Sesi | Hapus cookie sesi |
| GET | `/api/auth/me` | Sesi | User aktif |
| PATCH | `/api/auth/profile` | Sesi | Ubah profil sendiri |
| POST | `/api/auth/change-password` | Sesi | Ganti password + naikkan `tokenVersion` |

### Dashboard

| Method | Path | Akses | Fungsi |
| --- | --- | --- | --- |
| GET | `/api/dashboard/search` | Sesi | Pencarian global multi-entitas |
| GET | `/api/dashboard/notifications` | Sesi | Notifikasi tindak lanjut |
| POST | `/api/dashboard/notifications/read` | Sesi | Tandai sudah dibaca |

### Data Master

| Method | Path | Akses |
| --- | --- | --- |
| GET/POST | `/api/data-master/students` | GET: SUPERADMIN/OWNER/ADMIN/COACH · POST: SUPERADMIN/OWNER/ADMIN |
| PATCH/DELETE | `/api/data-master/students/[id]` | PATCH: SUPERADMIN/OWNER/ADMIN · DELETE: SUPERADMIN/OWNER |
| GET/POST | `/api/data-master/coaches` | SUPERADMIN/OWNER |
| PATCH/DELETE | `/api/data-master/coaches/[id]` | SUPERADMIN/OWNER |
| GET/POST | `/api/data-master/staff` | SUPERADMIN/OWNER |
| PATCH/DELETE | `/api/data-master/staff/[id]` | SUPERADMIN/OWNER |
| GET/POST | `/api/data-master/training-venues` | SUPERADMIN/OWNER |
| PATCH/DELETE | `/api/data-master/training-venues/[id]` | SUPERADMIN/OWNER |

### Operasional

| Method | Path | Akses | Fungsi |
| --- | --- | --- | --- |
| GET | `/api/operational/occurrences` | Sesi | Occurrence mingguan per program |
| POST | `/api/operational/occurrences/coach-assignment` | SUPERADMIN/OWNER/ADMIN | Assign pelatih |
| DELETE | `/api/operational/occurrences/coach-assignment` | SUPERADMIN/OWNER/ADMIN | Lepas pelatih |
| POST | `/api/operational/occurrences/private-booking` | SUPERADMIN/OWNER/ADMIN/STUDENT | Booking Private |
| DELETE | `/api/operational/occurrences/private-booking` | SUPERADMIN/OWNER/ADMIN/STUDENT | Batalkan booking |
| GET | `/api/operational/schedules` | Sesi | Daftar jadwal |
| POST | `/api/operational/schedules` | SUPERADMIN/OWNER/ADMIN | Buat jadwal |
| PATCH | `/api/operational/schedules/[id]` | SUPERADMIN/OWNER/ADMIN | Ubah jadwal |
| DELETE | `/api/operational/schedules/[id]` | SUPERADMIN/OWNER | Hapus jadwal |
| GET | `/api/operational/reports` | Sesi (terfilter role) | Daftar raport |
| POST | `/api/operational/reports` | SUPERADMIN/OWNER/ADMIN/COACH | Buat raport |
| PATCH | `/api/operational/reports/[id]` | SUPERADMIN/OWNER/ADMIN/COACH (miliknya) | Ubah / terbitkan |
| DELETE | `/api/operational/reports/[id]` | SUPERADMIN/OWNER | Hapus raport |

### Finance

| Method | Path | Akses | Fungsi |
| --- | --- | --- | --- |
| GET | `/api/finance/packages` | SUPERADMIN/OWNER/ADMIN | **Read-only** katalog untuk Finance |
| GET/POST | `/api/finance/transactions` | SUPERADMIN/OWNER/ADMIN | Daftar / buat transaksi |
| PATCH/DELETE | `/api/finance/transactions/[id]` | PATCH: SUPERADMIN/OWNER/ADMIN · DELETE: SUPERADMIN/OWNER | Ubah status/data / hapus |
| GET/POST | `/api/finance/subscriptions` | SUPERADMIN/OWNER/ADMIN | Daftar / buat langganan |
| PATCH/DELETE | `/api/finance/subscriptions/[id]` | PATCH: SUPERADMIN/OWNER/ADMIN · DELETE: SUPERADMIN/OWNER | Ubah / hapus |
| POST | `/api/finance/renewal-requests` | SUPERADMIN/STUDENT | Ajukan perpanjangan dari Paket Saya |

### Marketing

| Method | Path | Akses |
| --- | --- | --- |
| GET/POST | `/api/marketing/leads` | SUPERADMIN/OWNER |
| GET/PATCH | `/api/marketing/leads/[id]` | SUPERADMIN/OWNER |
| GET/POST | `/api/marketing/leads/[id]/activities` | SUPERADMIN/OWNER |
| GET/POST | `/api/marketing/articles` | SUPERADMIN/OWNER |
| GET/PATCH/DELETE | `/api/marketing/articles/[id]` | SUPERADMIN/OWNER |
| POST | `/api/marketing/articles/preview` | SUPERADMIN/OWNER (sanitasi HTML preview) |
| GET/POST | `/api/marketing/promotions` | SUPERADMIN/OWNER |
| POST | `/api/marketing/promotions/batch` | SUPERADMIN/OWNER (voucher batch) |
| GET/PATCH/DELETE | `/api/marketing/promotions/[id]` | SUPERADMIN/OWNER |
| GET/POST | `/api/marketing/packages` | SUPERADMIN/OWNER (pemilik katalog) |
| GET/PATCH/DELETE | `/api/marketing/packages/[id]` | SUPERADMIN/OWNER |

### Settings & Publik

| Method | Path | Akses | Fungsi |
| --- | --- | --- | --- |
| GET/PATCH | `/api/settings/system` | SUPERADMIN | Baca / simpan konfigurasi sistem |
| GET/POST | `/api/settings/backup` | SUPERADMIN | Daftar / buat backup |
| GET | `/api/settings/backup/[id]` | SUPERADMIN | Unduh file backup |
| POST | `/api/leads` | **Publik** | Pendaftaran lead dari landing page |

Format respons konsisten:

```json
// Sukses
{ "data": { ... } }

// Error
{ "code": "VALIDATION_ERROR", "message": "...", "fieldErrors": { "field": ["..."] } }
```

---

## 11. Fitur Lintas Modul

### 11.1 Pencarian global di topbar

- Komponen `GlobalSearch` + hook `useGlobalSearch` + service `searchDashboard`.
- Query minimal 2 karakter, di-debounce, maksimal 80 karakter.
- Mencari di: Siswa, Pelatih, Staff, Tempat Latihan, Jadwal, Raport, Paket, Langganan, Transaksi.
- Hasil dikelompokkan per entitas dengan navigasi keyboard; item mengarah ke halaman tujuan **beserta query** (`?q=`).
- Hasil sudah difilter role dan `programScope`; siswa tidak bisa menemukan data internal.

### 11.2 Notifikasi tindak lanjut

- Sumber: `notificationService` — bukan data dummy. Kunci notifikasi deterministik, misalnya `transaction-pending:<id>` atau `subscription-expiring:<id>:<expiresAt>`.
- Contoh notifikasi per role:
  - Manajemen: lead baru, pembayaran menunggu, paket segera berakhir, siswa menunggu aktivasi, draf raport.
  - COACH: kelas hari ini, draf raport, booking Private dengan siswanya.
  - STUDENT: akun menunggu aktivasi, tagihan menunggu, jadwal Private, raport terbit.
- Status "sudah dibaca" disimpan di tabel `NotificationRead` per user (unik per `userId + notificationKey`).
- Badge unread memakai **biru brand**, bukan merah, agar tidak terkesan alarm permanen.
- Auto-dismiss memakai `setTimeout` hanya untuk menutup popover/toast — tidak ada delay loading artifisial.

### 11.3 Pencarian per halaman dengan query string

Banyak halaman menerima `?q=` sehingga hasil pencarian global dapat langsung membuka halaman tujuan dengan filter aktif (Siswa, Finance, Raport, Jadwal).

### 11.4 Backup database

- `createDatabaseBackup()` mengekspor snapshot JSON isi database ke kolom `content`.
- Snapshot backup **tidak** ikut mengekspor isi backup lama agar ukuran tidak tumbuh rekursif.
- Tersedia daftar histori dan unduh file.
- Belum ada fitur **restore** otomatis.

### 11.5 Export CSV

- Finance Laporan dan Campaign menyediakan export CSV.
- Setiap sel lolos `escapeCsvCell()` sebelum ditulis.

---

## 12. Jaminan Integritas Data

Bagian ini merangkum keputusan teknis yang mencegah bug kelas berat.

### 12.1 Booking Private aman dari race condition

- `lockSlotForUpdate()` menjalankan `SELECT ... FOR UPDATE` pada baris `Slot` sebagai **operasi pertama** dalam transaksi.
- Semua transaksi yang mengubah slot yang sama mengantre, sehingga pola count → create tidak bisa saling mendahului pada snapshot REPEATABLE READ.
- Slot dengan status `CLOSED` atau `CANCELLED` ditolak (`assertSlotBookable`).
- `bookedCount` dan status slot (`OPEN`/`FULL`) diperbarui secara konsisten; pembatalan mengembalikan kuota.
- Kapasitas Private selalu 1.

### 12.2 Transisi pembayaran `PAID` exactly-once

Pada `updateTransaction()`:

1. Baris transaksi dan langganan dikunci (`FOR UPDATE`).
2. Transisi status diklaim dengan **CAS**:
   ```js
   const statusClaim = await tx.transaction.updateMany({
     where: { id, status: currentStatus },
     data: { status: data.status, paidAt: ... },
   });
   if (statusClaim.count !== 1) throw new FinanceError(409, 'TRANSACTION_STATE_CONFLICT', ...);
   ```
3. Efek finansial (perpanjangan langganan, pembuatan langganan baru, aktivasi siswa `PENDING → ACTIVE`) hanya dijalankan **setelah** klaim berhasil.
4. Transisi status tidak valid (mis. `CANCELLED → PAID`) ditolak dengan `409 INVALID_STATUS_TRANSITION`.

Hasil: klik ganda atau request paralel tidak dapat menggandakan masa aktif langganan.

### 12.3 Tanggal bisnis mengikuti WIB

- Tanggal "hari ini" untuk invoice, tanggal pembayaran, dan tanggal mulai langganan dihitung dengan `Asia/Jakarta`, bukan timezone server.
- Minggu operasional dimulai **Senin** (`getMondayDateKey`).
- Tanggal berbasis `@db.Date` dipisahkan dari timestamp agar tidak bergeser karena konversi zona waktu.

### 12.4 Sanitasi HTML artikel

- `sanitizeArticleHtml()` dijalankan **di server** sebelum disimpan dan sebelum ditampilkan.
- `sanitize-html` tidak pernah diimpor ke Client Component.
- Preview artikel (`/api/marketing/articles/preview`) juga melewati sanitasi.

### 12.5 Export CSV aman dari formula injection

`escapeCsvCell()`:

- Angka tetap dikirim sebagai angka.
- Teks yang karakter bermakna pertamanya `=`, `+`, `-`, atau `@` (termasuk setelah whitespace/zero-width) diberi prefiks apostrof agar dieksekusi sebagai teks.
- Cell dengan `"`, koma, `\r`, `\n` di-quote dan di-escape dengan benar.

### 12.6 Perlindungan PII

- STUDENT tidak pernah menerima `email` atau `username` pelatih; data pelatih dipangkas di service sebelum dikirim.
- `healthHistory` siswa dibatasi per role.
- Endpoint publik `/api/leads` tidak mengembalikan data lead.
- Raport `DRAFT` tidak pernah terekspos ke siswa/orang tua.

### 12.7 Keamanan sesi

- Token ditandatangani HMAC; `tokenVersion` memberi mekanisme pencabutan sesi.
- Password selalu bcrypt; tidak ada plaintext.
- Rate limit login persisten di database, tahan restart dan multi-instance.
- Kunci rate limit berupa hash, sehingga IP/username mentah tidak tersimpan.
- Cookie `httpOnly`, `sameSite=lax`, `secure` di production.

### 12.8 Validasi input dua lapis

- Skema Zod di `lib/validations/*` dipakai di server (Route Handler) **dan** client (untuk pesan error per field).
- Error API seragam: `code`, `message`, `fieldErrors`.

---

## 13. Design System & Perbaikan UI

### 13.1 Identitas visual

| Elemen | Nilai |
| --- | --- |
| Warna utama | Biru laut `#0066CC` |
| Warna dasar | Putih + skala slate |
| Font | Inter (body) + Manrope (aksen), via `next/font/google` |
| Sudut | `rounded-xl` s/d `rounded-3xl` |
| Bayangan | Halus (`shadow-sm`, `shadow-slate-900/[0.02]`) |
| Bahasa | Indonesia, nada profesional-ramah |

### 13.2 Warna program (sumber tunggal)

`lib/operational/constants.js` → `PROGRAM_CONFIG` + `getProgramBadgeTone()`:

| Program | Tone |
| --- | --- |
| AAA Sport Academy | `green` |
| A3 | `blue` |
| SWDC Swimmer | `violet` |
| Private | `amber` |

Helper menerima kode (`AAA`) maupun label (`SWDC Swimmer`), sehingga konsisten di Operasional, Data Siswa, Pelatih, Finance, dan Katalog Paket.

### 13.3 Komponen UI bersama

| Komponen | Fungsi |
| --- | --- |
| `components/ui/StatCard.jsx` | Kartu statistik kanonis; tone default biru netral, amber/merah hanya untuk kebutuhan tindakan; hover **hanya** bila punya `href` |
| `components/ui/PageHeader.jsx` | Header konten tanpa judul visual kedua (judul tetap satu di topbar) |
| `components/ui/AdvancedFilters.jsx` | Popover "Filter lainnya (n)" dengan badge aktif, Reset, Selesai, klik-luar, dan Escape |
| `components/ui/RowActionButton.jsx` | Aksi baris dengan label teks terlihat + label aksesibel lengkap |
| `components/ui/Table.jsx` | Tabel deklaratif responsif; meneruskan `column.label` sebagai `data-label` |
| `components/ui/Card.jsx` | Kartu dasar |
| `components/ui/Badge.jsx` | Badge dengan tone |
| `components/ui/Button.jsx` | Tombol dengan varian |

### 13.4 Hasil audit & perbaikan 14 issue UI

| # | Issue | Status akhir |
| --- | --- | --- |
| 1 | Topbar search & badge hardcoded | **Sebagian besar sudah tidak berlaku** — search, notifikasi, dan badge Leads sudah database-backed. Badge unread diubah ke biru brand. |
| 2 | Judul ganda & hierarki heading | **Diperbaiki** — `PageHeader` bersama; hanya satu `<h1>` (topbar); sapaan & judul kontekstual memakai `<h2>` |
| 3 | Active state route dengan query | **Diperbaiki** — `isActivePath` membandingkan pathname **dan** query; Finance/Settings akurat |
| 4 | Indikator/logout/tautan ganda, label sidebar | **Diperbaiki** — titik submenu, logout footer, label "Menu Utama", subteks redundan dihapus; Profil → `?tab=profile`; Pengaturan hanya untuk yang berhak |
| 5 | Tipografi terlalu berat & micro-label | **Diperbaiki** — `font-extrabold` → `font-bold`, header tabel/label `font-semibold`, micro-label uppercase dinormalisasi |
| 6 | Warna program/level tidak konsisten | **Diperbaiki** — dipusatkan di `getProgramBadgeTone()` |
| 7 | Banner read-only menyerupai peringatan | **Diperbaiki** — notice netral (ikon Info); amber hanya untuk peringatan/tindakan nyata; banner ScheduleCalendar tidak lagi bertumpuk |
| 8 | Kepadatan ikon aksi per baris | **Diperbaiki** — aksi berlabel teks pada Finance, Raport, Katalog, Data Master, Jadwal, Campaign, Content Hub |
| 9 | Save bar Settings | **Diperbaiki** — hanya SUPERADMIN + tab non-profil + ada perubahan; sticky mengikuti konten, tanpa offset hardcoded |
| 10 | StatCard belum terpusat | **Diperbaiki** — satu komponen kanonis; tone bermakna; hover lift pada kartu non-interaktif dihapus |
| 11 | Toolbar Finance & Jadwal terlalu padat | **Diperbaiki** — search + 2 filter utama, sisanya di popover dengan badge & Reset |
| 12 | Kontras teks kecil; delay loading buatan | **Kontras diperbaiki**; klaim `setTimeout` artifisial **stale** (tidak ditemukan) |
| 13 | Beranda mock/role switcher | **Stale** — beranda sudah memakai sesi & database; satu-satunya masalah (`h1` ganda) sudah diperbaiki |
| 14 | Tabel/kalender/form tidak ramah layar kecil | **Diperbaiki** — `.mobile-card-table`, agenda vertikal, form satu kolom hingga `md` |

### 13.5 Responsif

**Tabel menjadi kartu di mobile** — CSS `.mobile-card-table` di `app/globals.css` (aktif di bawah `768px`):

- `thead` disembunyikan secara visual (tetap ada untuk screen reader).
- Setiap `tr` menjadi kartu berbingkai; `tbody` menjadi grid bertumpuk.
- Setiap `td` menjadi dua kolom label–nilai; label diambil dari `data-label`.
- Sel Aksi menjadi blok penuh dengan tombol rata kiri.
- Diterapkan pada tabel Siswa, Pelatih, Staff, tiga tabel Finance, Raport, Jadwal Keseluruhan, dan `Table.jsx` bersama (Campaign).

**Kalender** — grid 7 kolom hanya di `lg+`; di bawahnya agenda vertikal per hari.

**Form modal** — grid dua kolom baru aktif mulai `md`, sehingga layar kecil tetap satu kolom.

### 13.6 Aksesibilitas

- Satu `<h1>` per halaman; hierarki heading konsisten.
- Tombol ikon memiliki `aria-label`.
- Popover/menu memakai pola `role="menu"`, `aria-expanded`, `aria-haspopup`.
- Skip link di landing page.
- Status loading memakai `role="status"` + teks `sr-only`.
- Kontras teks kecil dinaikkan ke `slate-500` (placeholder/disabled tetap lebih terang).
- Escape & klik-luar menutup popover.

---

## 14. Data Seed & Akun Demo

Seeder (`prisma/seed.js`) bersifat idempotent dan memakai `deleteMany`/upsert pada kunci unik.

### 14.1 Akun

| Role | Username | Password |
| --- | --- | --- |
| SUPERADMIN | `superadmin` | `password123` |
| OWNER | `owner` | `password123` |
| ADMIN AAA | `adminaaa` | `password123` |
| ADMIN A3 | `admina3` | `password123` |
| ADMIN SWDC | `adminswdc` | `password123` |
| COACH | `coach01` … `coach10` | `password123` |
| STUDENT | `student01` … `student10` | `password123` |

> Password di-hash dengan **bcryptjs** (cost 10). Ganti seluruh password seed sebelum dipakai di production.

### 14.2 Data lain

| Data | Jumlah / Keterangan |
| --- | --- |
| Tempat latihan (`Pool`) | 10 — Kolam Utama AAA (4), Kolam Kompetisi A3 (3), Kolam Atlet SWDC (2), Kolam Private Betikan (1, status MAINTENANCE) |
| Staff | 8 |
| Jadwal (`Schedule`) | 23 — AAA 5, A3 5, SWDC 5, dan 8 jadwal Private (kapasitas 1) |
| Katalog paket | 6 (AAA 1 & 3 bulan, A3 1 bulan, SWDC semester, Private single & 4 sesi) |
| Langganan | per siswa, dengan variasi ACTIVE/EXPIRED/PENDING dan sisa sesi |
| Raport | beberapa siswa, campuran DRAFT dan PUBLISHED |
| Lead | 7 dengan variasi status, sumber, dan aktivitas |
| Promosi | 3 (termasuk kode referral `SAHABATAAA`) |
| Artikel | 3 (status bervariasi) |
| Testimoni | 3 |
| FAQ | 5 |
| Settings sistem | 1 baris `id = "default"` + template notifikasi |

Avatar seed memakai UI Avatars; gambar hero memakai file lokal `public/images/hero-swimming.jpg`.

---

## 15. Validasi yang Sudah Dilakukan

### 15.1 Validasi fungsional (riwayat pengembangan)

| Uji | Hasil |
| --- | --- |
| Prisma 6.19.3 terhadap MariaDB `aaa_test` | PASS |
| Migrasi/sinkronisasi schema | PASS |
| Seeder idempotent (dijalankan berulang) | Stabil |
| Suite tiruan + esbuild | PASS |
| Build salinan E2E | **49/49 PASS** |

### 15.2 Validasi batch perbaikan UI 14 issue

| Pemeriksaan | Hasil |
| --- | --- |
| Esbuild syntax check seluruh file JS/JSX berubah (41 file) | `ESBUILD SYNTAX CHECK PASSED` |
| `node --check` untuk menu, hook, dan konstanta | `NODE SYNTAX CHECK PASSED` |
| Harness active-state query-aware (10 kasus) | `QUERY-AWARE ACTIVE PATH TESTS PASSED` |
| Harness tone program (kode + label) | `PROGRAM BADGE TONE TESTS PASSED` |
| Inspeksi layout dashboard (Suspense + `instant`) | Sesuai (boundary aman untuk `useSearchParams`) |

### 15.3 Panduan uji manual yang disarankan

1. Login `adminaaa` → pastikan tidak melihat menu Pelatih/Staff/Tempat Latihan/Marketing/Settings sistem, dan hanya melihat program AAA.
2. Login `coach01` → coba ubah raport milik pelatih lain (harus ditolak).
3. Login `student02` (raport DRAFT) → raport harus tidak terlihat.
4. Login `student01` → booking slot Private kosong dari kalender, lalu buka di dua tab dan klik serentak (harus hanya satu yang berhasil).
5. Finance: konfirmasi satu transaksi `PENDING` dua kali cepat (masa aktif langganan tidak boleh bertambah dua kali).
6. Marketing: ubah artikel berisi `<script>` → HTML harus tersanitasi.
7. Finance Laporan: export CSV berisi sel diawali `=` → harus terbaca sebagai teks.
8. Buka dashboard di lebar 375px → tabel menjadi kartu, kalender menjadi agenda, tidak ada scroll horizontal.
9. Settings: buka tab Profil → save bar tidak boleh muncul.

---

## 16. Status & Pekerjaan Lanjutan

### 16.1 Sudah selesai

- Landing page penuh berbasis database dengan cache tag & revalidasi.
- Autentikasi username + bcrypt + sesi HMAC + `tokenVersion` + rate limit persisten.
- RBAC lima role dengan scope program, ditegakkan di menu, halaman, API, dan query.
- Data Master (Siswa, Pelatih, Staff, Tempat Latihan) dengan CRUD sesuai role.
- Operasional: jadwal berulang, occurrence mingguan, penugasan pelatih, booking Private aman race condition, kalender per program.
- Raport: kompetensi per level, penilaian, publikasi, kenaikan level, cetak, kepemilikan penilai.
- Finance: transaksi, langganan, laporan, pengingat WhatsApp, export CSV, exactly-once saat pelunasan.
- Paket Saya: langganan, riwayat, pengajuan perpanjangan.
- Marketing: katalog paket, campaign/promosi/voucher/referral, content hub tersanitasi, CRM lead dengan aktivitas.
- Settings: profil & keamanan, general, financial rules (konfigurasi), template notifikasi, backup JSON.
- Pencarian global & notifikasi berbasis database.
- Audit dan perbaikan 14 issue UI termasuk responsif mobile.
- Dokumen teknis per perbaikan (lihat §17).

### 16.2 Belum selesai / perlu tindak lanjut

| Item | Keterangan | Dampak bila dilewati |
| --- | --- | --- |
| **Terapkan `users.token_version`** | Kolom sudah ada di schema, perlu `npm run db:push` atau migration ke database target | Login gagal / skema tidak cocok |
| **Folder `prisma/migrations`** | Belum ada; saat ini memakai `db:push` | Belum reproducible untuk deploy tim |
| **`eslint.config.*`** | Belum dibuat, `npm run lint` belum bisa jalan | Kualitas kode tidak terjaga otomatis |
| **`next.config.*`** | Belum ada; konfigurasi Cache Components belum eksplisit di file | Perlu ditetapkan sebelum deploy |
| **Full build & E2E di environment nyata** | Batch UI terakhir hanya diverifikasi dengan esbuild + node syntax, belum `next build` dan uji browser penuh | Risiko regresi visual belum tertutup penuh |
| **Financial Rules sebagai rule engine** | Konfigurasi pajak/denda tersimpan, tetapi belum otomatis diterapkan ke invoice | Perhitungan pajak/denda masih manual |
| **Restore backup** | Baru tersedia buat/daftar/unduh | Pemulihan data masih manual |
| **TanStack Query** | Terpasang tetapi belum dipakai untuk data fetching | Belum ada cache/mutasi otomatis di client |
| **Google Drive / integrasi eksternal** | Belum ada integrasi penyimpanan eksternal | Gambar disimpan sebagai URL/data URL di database |

### 16.3 Catatan sebelum production

1. Ganti `AUTH_SECRET` dengan nilai acak panjang.
2. Ganti seluruh password akun seed.
3. Set `NODE_ENV=production` agar cookie `secure` aktif (wajib HTTPS).
4. Terapkan `prisma migrate deploy` (setelah migration dibuat) alih-alih `db:push`.
5. Pastikan `DATABASE_URL` memakai user MySQL dengan hak minimum yang dibutuhkan.
6. Aktifkan backup terjadwal di level database/infrastruktur, bukan hanya lewat dashboard.

---

## 17. Dokumen Terkait

Dokumen teknis per topik yang sudah ada di root proyek:

| File | Isi |
| --- | --- |
| `RINGKASAN_PROYEK.md` | **Dokumen ini** — rangkuman menyeluruh |
| `SETUP_DEPENDENCIES.md` | Panduan dependensi dan setup |
| `AUTH_LOGIN_SECURITY_FIX.md` | Perbaikan keamanan login (rate limit, dummy hash) |
| `PROGRAM_ADMIN_SCOPE_IMPLEMENTATION.md` | Implementasi scope program untuk ADMIN |
| `RBAC_OWNER_IMPLEMENTATION.md` | Matriks role dan penambahan OWNER |
| `DATA_SISWA_IMPLEMENTATION.md` | Implementasi Data Master Siswa |
| `COACH_PII_STUDENT_EXPOSURE_FIX.md` | Pencegahan kebocoran PII pelatih ke siswa |
| `COACH_REPORT_OWNERSHIP_FIX.md` | Kepemilikan raport oleh pelatih penilai |
| `STUDENT_DRAFT_REPORT_VISIBILITY_FIX.md` | Draf raport tidak terekspos ke siswa |
| `RAPORT_SISWA_IMPLEMENTATION.md` | Fitur raport siswa |
| `SLOT_BOOKING_RACE_FIX.md` | Pengamanan race condition booking slot |
| `PAYMENT_RACE_CONDITION_FIX.md` | Exactly-once transisi pembayaran `PAID` |
| `PRINT_POPUP_BLOCKED_FIX.md` | Perbaikan deteksi popup cetak |
| `FINANCE_IMPLEMENTATION.md` | Modul Finance (Transaksi/Langganan/Laporan) + kepemilikan katalog di Marketing |
| `DASHBOARD_SHELL_FEATURES_IMPLEMENTATION.md` | Shell, menu, pencarian, notifikasi, tema |
| `MOCK_DATA_AUDIT.md` | **Dokumen historis** — audit mock data saat migrasi ke full database |

---

## Ringkasan dalam Satu Paragraf

AAA Sport Academy adalah aplikasi Next.js 16 App Router berbahasa JavaScript dengan Tailwind CSS v4 dan Prisma + MySQL. Pengunjung melihat landing page yang seluruhnya bersumber dari database — program, promo, testimoni, blog, dan FAQ — lalu dapat mendaftar melalui formulir yang langsung menjadi Lead di CRM. Pengguna terautentikasi masuk ke dashboard dengan lima tingkat role ber-scope program. Di dalamnya tersedia Data Master, Operasional (jadwal, booking Private, raport), Finance (transaksi, langganan, laporan), Paket Saya untuk siswa, Marketing (katalog paket, campaign, content hub, leads), dan Settings sistem. Integritas data dijaga melalui row lock + CAS (booking & pembayaran), sanitasi HTML di server, proteksi CSV injection, tanggal bisnis WIB, pencabutan sesi lewat `tokenVersion`, dan pemisahan data sensitif per role. UI sudah melewati audit 14 issue: satu `h1` per halaman, active state sadar query string, StatCard dan header bersama, aksi berlabel, filter lanjutan berbasis popover, serta tabel/kalender/form yang ramah layar kecil.
