# Architecture: Admin Panel

## Tech Stack
- Framework: Laravel 12
- Frontend: Blade + Tailwind CSS v4 + Livewire 3 + Alpine.js
- UI Library: **daisyUI v5** (component library dengan built-in themes)
- Animasi: **AOS** (Animate On Scroll) untuk landing page
- Icons: **Heroicons** via Blade Heroicons package
- Database: **SQLite (default dev)** → PostgreSQL untuk production
- Auth: Manual auth (session-based) dengan middleware

## Konsep Livewire

Navigasi menggunakan **Livewire full-page component** agar tidak ada refresh:
- Sidebar = Livewire component dengan state (`level`, `currentModule`)
- Konten = Livewire component yang berubah berdasarkan state sidebar
- Komunikasi antar komponen via event (`moduleSelected`, `pageSelected`, `back`)

## Struktur Routing

```
localhost/
├── /                    → Landing page (guest) — Blade biasa
├── /register            → Registrasi (tanpa auth) — Livewire component
├── /login               → Login — Livewire component
├── /logout              → Logout
├── /internal/{module?}/{page?}
│   └── Middleware: auth + isInternal
│   └── Livewire: Internal\Dashboard (full-page)
├── /eksternal/{module?}/{page?}
│   └── Middleware: auth + isEksternal
│   └── Livewire: Eksternal\Dashboard (full-page)
└── /admin/{section?}/{action?}
    └── Middleware: auth + superadmin
    └── Livewire: Admin\Dashboard (full-page)
```

## Sidebar Navigation (Livewire)

Sidebar component memiliki state internal:

```
state: { level: 1|2, currentModule: null|string, currentPage: null|string }

Level 1 (default):
┌──────────────────┐
│ ◼ HRIS           │──wire:click="selectModule('hris')"
│ ◼ Marketing      │
│ ◼ Finance        │
│ ◼ Eproc          │
└──────────────────┘

Level 2 (setelah klik module):
┌──────────────────┐
│ ← Kembali        │──wire:click="back()"
│──────────────────│
│ ◼ Data Pegawai   │──wire:click="selectPage('pegawai')"
│ ◼ Absensi        │
│ ◼ Penggajian     │
└──────────────────┘
```

Alur navigasi (tanpa refresh):
1. User klik module → `selectModule('hris')` → sidebar state level=2, konten render ModuleGrid
2. User klik sub-menu → `selectPage('pegawai')` → konten render halaman pegawai
3. User klik "Kembali" → `back()` → sidebar state level=1, konten render dashboard
4. URL ikut berubah via `$this->redirect()` atau history.pushState

## Database Schema (Core)

### users
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | string | |
| email | string unique | |
| password | string (hashed) | |
| email_verified_at | timestamp | nullable |
| role_id | FK → roles.id | nullable (sampai di-approve) |
| is_approved | boolean | default false |
| is_active | boolean | default true |
| is_eksternal | boolean | redundant dari role |
| session_id | string | nullable — untuk single session |
| last_login_at | timestamp | nullable |
| last_login_ip | string | nullable |
| created_by | FK → users.id | nullable |
| updated_by | FK → users.id | nullable |
| created_at | timestamp | |
| deleted_at | timestamp | soft delete |

### modules
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | string | nama modul (HRIS, Marketing, Finance, Eproc) |
| slug | string unique | slug untuk routing (hris, marketing, finance, eproc) |
| icon | string | icon class sidebar |
| description | text | nullable |
| is_eksternal | boolean | true = hanya untuk user eksternal |
| order | integer | urutan di sidebar |
| is_active | boolean | |

Module = entitas terpisah dari menu. Module adalah kategori besar (Level 1 sidebar).

### roles
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | string | superadmin, admin, user, vendor |
| is_eksternal | boolean | false = internal, true = eksternal |

### menus
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| module_id | FK → modules.id | module tempat menu ini berada |
| name | string | label sidebar |
| route | string | route name |
| icon | string | icon class |
| parent_id | FK → menus.id (nullable) | parent menu (untuk sub-sub-menu) |
| order | integer | urutan |
| is_active | boolean | |

**Logic**: Module = Level 1 sidebar. Menu dengan `module_id` = sub-menu (Level 2). Menu dengan `parent_id` = sub-sub-menu (jika ada).

### permissions
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| role_id | FK → roles.id | |
| menu_id | FK → menus.id | |
| can_create | boolean | default false |
| can_read | boolean | default true |
| can_update | boolean | default false |
| can_delete | boolean | default false |
| can_approve | boolean | default false |

**Constraint**: unique (role_id, menu_id)

## Middleware
1. **CheckInternal** — user login + role->is_eksternal == false
2. **CheckEksternal** — user login + role->is_eksternal == true
3. **CheckSuperAdmin** — user login + role->name == superadmin

## Data Flow
```
Guest → Landing Page

Register (Livewire Auth\Register):
  → create user (is_approved=false, role_id=null, email_verified_at=null)
  → kirim email verifikasi (di dev: tampilkan link di flash message)
  → redirect ke login dengan notif "Cek email untuk verifikasi"

Verify (route /verify-email/{token}):
  → set email_verified_at = now
  → redirect ke login: "Email berhasil diverifikasi, tunggu approval admin"

Login (Livewire Auth\Login):
  → validasi email+password
  → email_verified? true:lanjut false:tolak "Verifikasi email dulu"
  → is_approved? true:lanjut false:tolak "Tunggu approval admin"
  → is_active? true:lanjut false:tolak "Akun dinonaktifkan"
  → single session: invalidate session lama jika ada
  → update last_login_at, last_login_ip, session_id
  → log IP anomaly jika dari IP baru
  → redirect ke /internal/dashboard atau /eksternal/dashboard

Dashboard (Livewire Internal\Dashboard):
  → mount($module, $page)
  → Sidebar: sidebar.blade.php dengan state dari component
  → Konten: @switch berdasarkan module & page

Navigasi:
  wire:click → update state → re-render sidebar & konten (no refresh)
  redirect → kalau perlu ganti URL (bookmark-able)
```

## Frontend Design

### Design System
- **daisyUI theme**: gunakan theme "dark" atau "corporate" untuk admin panel
- **Landing page**: theme "light" dengan aksen gradient
- **Font**: Inter (sans-serif) via Google Fonts
- **Icons**: Heroicons (outline untuk sidebar, solid untuk CTA)

### Landing Page (Public)
Landing page adalah entry point untuk guest. Desain modern dengan komponen:

```
┌─────────────────────────────────────────────────┐
│ [Logo]  Beranda  Fitur  Tentang  Kontak  |Login│ ← Navbar sticky
│─────────────────────────────────────────────────│
│                                                 │
│   Hero Section: headline besar + CTA + ilus-    │
│   trasi / gradient background dengan animasi    │
│                                                 │
│   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐          │
│   │Icon │  │Icon │  │Icon │  │Icon │          │ ← Features grid
│   │Fitur1│  │Fitur2│  │Fitur3│  │Fitur4│          │   (AOS fade-up)
│   └─────┘  └─────┘  └─────┘  └─────┘          │
│                                                 │
│   About Us section dengan stats counter          │
│   (500+ users, 4 modul, 99.9% uptime)          │
│                                                 │
│   Contact form dengan card modern                │
│                                                 │
│   Footer: 4 kolom, links, social icons          │
└─────────────────────────────────────────────────┘
```

### Admin Panel (Internal/Eksternal)
```
┌─────────────────────────────────────────────────┐
│ ☰ Logo          [Search]  🔔  👤 Admin          │ ← Navbar
├──────────┬──────────────────────────────────────┤
│Sidebar   │                                      │
│◼ HRIS    │   Dashboard / Content Area           │
│◼ Market. │   - Cards dengan stats               │
│◼ Finance │   - Tabel modern                      │
│◼ Eproc   │   - Grafik / chart                   │
│          │                                      │
│◼ ⚙ Admin │                                      │
└──────────┴──────────────────────────────────────┘
```

### Auth Pages (Login & Register)
Halaman auth didesain sebagai **split-screen modern** — tidak seperti form login biasa.

```
Desktop:                              Mobile:
┌──────────────┬───────────────┐      ┌─────────────────┐
│              │               │      │  [Logo]          │
│  Ilustrasi   │   Logo kecil  │      │                 │
│  / Branding  │   👋 Selamat  │      │ ┌─────────────┐ │
│  Carousel    │   Datang      │      │ │ Email       │ │
│  testimoni   │               │      │ │ Password    │ │
│              │ ┌───────────┐ │      │ │ [Login]     │ │
│              │ │ Email     │ │      │ │ Register →  │ │
│              │ │ Password  │ │      │ └─────────────┘ │
│              │ │ [Login]   │ │      │                 │
│              │ │ Register→ │ │      │ © 2025          │
│              │ └───────────┘ │      └─────────────────┘
│              │               │
│              │ © 2025        │
└──────────────┴───────────────┘
```

**Elemen Desain:**
- Split-screen: kiri = brand illustration / gradient / testimonial carousel, kanan = form
- Card form: glassmorphism (background blur + transparan) atau daisyUI card dengan shadow
- Background: gradient dari logo perusahaan atau ilustrasi abstrak
- Logo perusahaan di atas form
- Social proof: "Digunakan oleh 500+ perusahaan" di bawah form
- Mobile: fullscreen form saja, ilustrasi dihilangkan
- Animasi: fade-in form on load (Alpine.js x-transition)
- Error/success notification: toast modern di pojok kanan atas (daisyUI alert + Alpine.js auto-dismiss)

### Form Components Library
Semua input form menggunakan library modern agar terlihat atraktif dan interaktif, bukan HTML biasa.

| Komponen | Library | Fitur |
|---|---|---|
| **Text Input** | daisyUI `input` + Alpine.js | Floating label, icon prefix/suffix, char counter, auto-focus |
| **Select (biasa)** | daisyUI `select` | Styling sesuai tema |
| **Select (searchable)** | **Tom Select** | Search, tag, multiple, remote data, creatable |
| **Textarea** | daisyUI `textarea` + Alpine.js | Auto-resize, karakter counter |
| **Datepicker** | **flatpickr** | Kalender interaktif, min/max date, disable dates, range, inline |
| **Time picker** | **flatpickr** (`enableTime`) | Pilih jam:menit, format 24/12 jam |
| **DateTime picker** | **flatpickr** (`enableTime` + `dateFormat`) | Kombinasi tanggal + jam |
| **File Upload** | **FilePond** | Drag & drop, preview, multiple, resize, image crop, progress bar |
| **Rich Text** | **Trix Editor** | Bold, italic, list, link, heading — ringan & terintegrasi |

**Integrasi Livewire:**
Semua library diinisialisasi via **Alpine.js `x-init`** agar tetap reaktif dengan Livewire:

```blade
<div x-data x-init="flatpickr($refs.dateInput, { dateFormat: 'Y-m-d' })">
    <input x-ref="dateInput" type="text" wire:model="tanggal"
           class="input input-bordered w-full" />
</div>

<div x-data x-init="new TomSelect($refs.select, { create: true })">
    <select x-ref="select" wire:model="user_id"
            class="select select-bordered w-full">
        <option value="1">User A</option>
    </select>
</div>

<div x-data x-init="FilePond.create($refs.filepond, { allowMultiple: true })">
    <input x-ref="filepond" type="file" wire:model="files" multiple />
</div>
```

**Styling:**
- Semua input menggunakan daisyUI class: `input-bordered`, `select-bordered`, `textarea-bordered`
- Floating label: kombinasi `label` + `input` daisyUI
- Error state: `input-error`, `select-error`, `textarea-error`
- FilePond di-theme ulang pakai daisyUI warna (custom CSS minimal)
- flatpickr di-theme pakai `flatpickr-airbnb` atau custom CSS sesuai daisyUI theme
- Tom Select: theme `tom-select.css` atau custom

**Loading & Validation:**
- Semua form input tetap mengikuti aturan Loading States (spinner di button submit)
- Validasi error muncul di bawah input dengan `text-red-500` + icon
- Disabled state: `input-disabled` saat processing

### Maps — Interaktif & Poligon
Menggunakan **Leaflet.js** (gratis, open-source, tanpa API key) + **Leaflet Draw** untuk poligon.

| Fitur | Library |
|---|---|
| Peta dasar | Leaflet + OpenStreetMap tiles |
| Poligon wilayah | **Leaflet Draw** (draw, edit, delete polygon) |
| GeoJSON Indonesia | Data batas provinsi/kabupaten dari **geojson-indonesia** area |
| Marker & Popup | Leaflet native |
| Choropleth (heatmap wilayah) | Leaflet + plugin choropleth |

**Integrasi Livewire:**
```blade
<div x-data x-init="map = L.map($refs.map).setView([-2.5, 117], 5);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
// Load GeoJSON Indonesia
fetch('/geojson/provinsi.json').then(r => r.json()).then(data => {
    L.geoJSON(data, { style: { color: '#2563eb' } }).addTo(map);
});
// Polygon draw
draw = new L.Control.Draw({ edit: { featureGroup: drawnItems } });
map.addControl(draw);">
    <div x-ref="map" class="h-[500px] w-full rounded-xl border"></div>
</div>
```

Alternatif premium (untuk produksi): **Mapbox GL JS** — 3D, custom style, lebih halus.

### Charts — Visual Data
Menggunakan **ApexCharts** — paling atraktif, animasi smooth, interaktif, dan gratis.

| Tipe Chart | Penggunaan |
|---|---|
| **Line / Area** | Tren data per periode |
| **Bar (vertical/horizontal)** | Perbandingan antar kategori |
| **Pie / Donut** | Distribusi / komposisi |
| **Radial Bar** | Progress / persentase |
| **Heatmap** | Kalender aktivitas |
| **Mixed** | Kombinasi line + bar |

**Integrasi Livewire:**
```blade
<div x-data="{
    chart: null,
    init() {
        this.chart = new ApexCharts($refs.chart, {
            series: [{ name: 'Pegawai', data: @js($series) }],
            chart: { type: 'bar', height: 350, toolbar: { show: true } },
            colors: ['#2563eb'],
            plotOptions: { bar: { borderRadius: 8, horizontal: false } },
            xaxis: { categories: @js($categories) }
        });
        this.chart.render();
    },
    // Update real-time via Livewire event
    @entangle('series')
}">
    <div x-ref="chart" class="w-full"></div>
</div>
```

### PDF Export
Menggunakan **barryvdh/laravel-dompdf** — generate PDF dari Blade view.
- Export halaman / laporan ke PDF
- Styling dengan CSS (Tailwind support via dompdf)
- Download otomatis atau simpan ke storage
- Composer: `composer require barryvdh/laravel-dompdf`

### Excel Import / Export
Menggunakan **maatwebsite/laravel-excel** (Laravel Excel).
- Export data ke format: `.xlsx`, `.csv`
- Import dari file `.xlsx`, `.csv` langsung ke Collection/Model
- Batch import dengan queue untuk file besar
- Composer: `composer require maatwebsite/laravel-excel`

### Animasi & Interaksi
- **AOS**: fade-up, fade-left, zoom-in untuk landing page sections
- **Tailwind transitions**: hover scale, shadow, color transitions
- **Alpine.js `x-transition`**: modal, alert, sidebar toggle di admin
- **Livewire loading states**: `wire:loading` untuk spinner/loading skeleton

### Komponen Reusable
Semua komponen UI dibuat menggunakan **Blade components**:
- `<x-button>` — primary, secondary, outline, ghost
- `<x-card>` — card with header/body/footer
- `<x-input>` — input dengan label + error
- `<x-alert>` — success, error, warning, info
- `<x-modal>` — konfirmasi / form modal
- `<x-badge>` — status badge
- `<x-table>` — table dengan sort & search
- `<x-spinner>` — loading spinner reusable

### Loading States (Wajib di Semua Interaksi)
Setiap proses backend WAJIB menampilkan indikator loading. Tidak boleh ada proses yang langsung tanpa feedback.

**Pola Livewire `wire:loading`:**
```blade
<button wire:click="save" wire:loading.attr="disabled">
    <span wire:loading.remove>Simpan</span>
    <span wire:loading>
        <x-spinner /> Menyimpan...
    </span>
</button>
```

**Tipe Loading:**
| Komponen | Target | Visual |
|---|---|---|
| Button submit | `wire:target` pada button | Spinner + teks "Menyimpan..." → button disabled |
| Daftar / tabel | `wire:target` pada method load | Skeleton/shimmer row (3-5 baris abu-abu animasi) |
| Sidebar navigasi | `wire:loading` pada sidebar | Sidebar tetap, konten area overlay spinner |
| Modal / form | `wire:target` pada method | Button disabled + spinner inline |
| Halaman penuh | `wire:loading` pada wrapper | Overlay spinner di tengah konten |
| Login / Register | Form submit | Button disabled + "Memproses..." |

**Aturan:**
- Semua `wire:click` → `wire:loading.attr="disabled"` + spinner di button
- Semua tabel/daftar → skeleton loading saat fetch data
- Gunakan `wire:target` untuk tepat sasaran (jangan loading seluruh halaman)
- Loading tidak boleh lebih dari 3 detik tanpa fallback "terjadi kesalahan"
- Animasi loading: daisyUI loading spinner (loading-spinner / loading-ring) atau Tailwind CSS animate-pulse untuk skeleton

### Login Throttling
- **5 attempts per minute** per kombinasi email + IP
- Menggunakan Laravel `RateLimiter` dengan key: `login|{email}|{ip}`
- Jika limit tercapai → tampilkan error notification di form login (bukan redirect)
- Notification: inline alert dengan Tailwind CSS (warna merah/red-500)
- Throttle di-reset setelah 1 menit

### Password Policy
- Minimal 8 karakter
- Wajib kombinasi: huruf + angka
- Hash bcrypt (default Laravel)
- Implementasi: `Illuminate\Validation\Rules\Password::min(8)->letters()->numbers()`

### Password Strength Indicator
- Real-time indicator di form registrasi menggunakan Alpine.js
- Visual: progress bar merah → kuning → hijau
- Kriteria: panjang (8+), huruf besar, huruf kecil, angka, simbol
- Implementasi: Alpine.js `x-data` + `x-model` pada input password

### Email Verification
- Setelah registrasi, kirim link verifikasi ke email user
- User klik link → `email_verified_at` terisi
- Admin hanya bisa approve user yang sudah verified
- Di development: tampilkan link verifikasi di flash message (skip mail driver)

### Single Session
- Setiap login sukses → simpan `session_id` ke tabel users
- Cek di middleware: jika session_id user != current session → force logout
- User yang di-force logout dapat notifikasi: "Akun Anda login dari perangkat lain"
- Guard: boot middleware di route group internal/eksternal

### IP Anomaly Detection
- Simpan riwayat IP login di log (last 5 unique IPs)
- Jika login dari IP baru (tidak ada di riwayat):
  - Catat sebagai `Log::info('Login from new IP', ...)`
  - Tampilkan notifikasi di dashboard: "Login dari IP baru: {ip}"

### Last Login Info
- Setiap login sukses → update `last_login_at` dan `last_login_ip`
- Tampilkan di dashboard: "Terakhir login: {tanggal} dari IP {ip}"

### Session Timeout
- Session lifetime: 120 menit idle
- Setelah timeout → redirect ke login dengan pesan "sesi berakhir"
- Konfigurasi di `config/session.php` → `lifetime`

### Audit Trail (Auth Events)
- Catat ke log: failed login attempt (email + IP + timestamp)
- Catat ke log: successful login (user_id + IP + timestamp)
- Catat ke log: logout (user_id + timestamp)
- Catat ke log: login dari IP baru (warning level)
- Gunakan `Log::channel('stack')->warning()` untuk failed attempt
- Untuk production: bisa upgrade ke tabel `activity_logs` terpisah

## Layering Architecture

### 3 Layer — Separation of Concerns
```
┌──────────────────────────────────────────────────────────────┐
│  VIEW LAYER  (resources/views/)                              │
│  Blade & Livewire templates — Hanya HTML, CSS, Alpine.js     │
│  └─ Tidak ada logic bisnis. Hanya $this-> dan @json.        │
├──────────────────────────────────────────────────────────────┤
│  CONTROLLER LAYER  (app/Livewire/ + app/Http/Controllers/)   │
│  Livewire Component — Handle input, validasi, event dispatch │
│  └─ Delegasi logic ke Service. Tidak ada query SQL.         │
├──────────────────────────────────────────────────────────────┤
│  SERVICE LAYER  (app/Services/)                              │
│  Business Logic — CRUD, kalkulasi, validasi bisnis, transaksi│
│  └─ Pure PHP. Tidak tahu HTTP, request, atau session.       │
├──────────────────────────────────────────────────────────────┤
│  MODEL LAYER  (app/Models/)                                  │
│  Eloquent — Query, relationship, scope, accessor, mutator    │
│  └─ Hanya data mapping. Tidak ada logic bisnis.             │
└──────────────────────────────────────────────────────────────┘
```

### Alur Request
```
Route → Middleware → Livewire Component
  → Validasi Request
  → Service::method($data)
    → Transaction DB
    → Activity Log
    → Return Result
  ← Component set property / flash message
← View re-render
```

### Module per Folder
Setiap modul punya folder sendiri di setiap layer agar programmer fokus:

| Layer | Path |
|---|---|
| View | `resources/views/modules/{hris,marketing,finance,eproc}/` |
| Livewire | `app/Livewire/Modules/{Hris,Marketing,Finance,Eproc}/` |
| Service | `app/Services/Modules/{Hris,Marketing,Finance,Eproc}/` |
| Model | `app/Models/Modules/{Hris,Marketing,Finance,Eproc}/` |

### Aturan Ketat
1. **Livewire** → hanya handling input, validasi, delegasi ke Service
2. **Service** → pure business logic, transaksi DB, logging
3. **Model** → hanya Eloquent (query, relationship, scope)
4. **View** → hanya blade rendering
5. **TIDAK BOLEH**: `DB::` atau Eloquent langsung di Livewire
6. **TIDAK BOLEH**: Business logic di dalam Model

## Laravel Pitfalls & Mitigations

### 1. N+1 Query pada Relasi Bertingkat
**Problem**: ERP punya relasi dalam (Module → Menu → Permission → Role → User). Tanpa eager loading, query bisa 100+ per halaman.

**Mitigasi**:
- Laravel strict mode: `Model::preventLazyLoading()` di `AppServiceProvider`
- Global scope: `Model::with()` default pada relasi yang selalu dipakai
- Wajib: `->with(['relasi.child'])` untuk setiap query di Service
- Logger: jika ada N+1 terdeteksi, langsung notifikasi developer

```php
// AppServiceProvider.php
Model::preventLazyLoading(! app()->isProduction());
```

### 2. Report Aggregat (SUM, GROUP BY, PIVOT)
**Problem**: Eloquent tidak cocok untuk report ERP kayak "total penjualan per bulan per cabang".

**Mitigasi**:
- Service Layer boleh pakai `DB::raw()` dan `DB::select()` — ini satu-satunya pengecualian
- Buat folder khusus: `app/Services/Modules/{Modul}/ReportService.php`
- Report tetap di Service layer, bukan di Livewire
- Gunakan Query Builder, bukan raw string SQL

```php
class LaporanKeuanganService {
    public function labaRugi($bulan, $tahun): Collection
    {
        return DB::table('transaksis')
            ->select(DB::raw("
                SUM(CASE WHEN tipe = 'pendapatan' THEN jumlah ELSE 0 END) as pendapatan,
                SUM(CASE WHEN tipe = 'biaya' THEN jumlah ELSE 0 END) as biaya,
                SUM(CASE WHEN tipe = 'pendapatan' THEN jumlah ELSE -jumlah END) as laba_rugi
            "))
            ->whereMonth('tanggal', $bulan)
            ->whereYear('tanggal', $tahun)
            ->get();
    }
}
```

### 3. Long Process (Export PDF/Excel, Payroll, Batch)
**Problem**: Export ribuan data atau kalkulasi payroll bikin request timeout.

**Mitigasi**:
- Semua long process wajib lewat **Laravel Queue** (job)
- Job dipisah per tipe: `app/Jobs/Modules/{Modul}/ExportPegawaiJob.php`
- Notifikasi user saat job selesai via Livewire event
- Progress bar di frontend dengan polling job status

```php
class ExportPegawaiJob implements ShouldQueue {
    use Dispatchable, InteractsWithQueue;

    public function handle(PegawaiService $service): void
    {
        $filePath = $service->exportExcel(request()->all());
        // Notifikasi user via database notification
        Notification::send($this->user, new ExportReadyNotification($filePath));
    }
}
```

### 4. Memory Footprint
**Problem**: Laravel ~40-60MB per request. Untuk ERP concurrent 500+ user, butuh RAM besar.

**Mitigasi**:
- Optimasi Queue Worker: `php artisan queue:work --queue=high,default --memory=256`
- Hindari `::all()` tanpa pagination — wajib paginate untuk list data
- Cache config & route di production: `php artisan optimize`
- Gunakan `chunk()` untuk loop data besar

### 5. Form Kompleks (50+ field, interconnected)
**Problem**: Livewire component jadi gemuk kalau 1 form = 50 field + validasi + logic.

**Mitigasi**:
- Split ke child Livewire components per section
- Dispatch event antar component
- Form hanya sebagai orchestrator, logic tetap di Service

```blade
{{-- Parent form --}}
<livewire:modules.hris.pegawai-form-data-pribadi />
<livewire:modules.hris.pegawai-form-kontrak />
<livewire:modules.hris.pegawai-form-dokumen />
```

### 6. Migration File Menumpuk
**Problem**: ERP 200+ tabel = 200+ migration file. Sulit dilacak.

**Mitigasi**:
- Tiap rilis: `php artisan schema:dump` → squash migrations jadi 1 file SQL
- Atau: satu migration per modul (`create_hris_tables.php`)
- Migration hanya untuk struktur awal, perubahan pakai migration baru

### 7. Multi-Tenant (nanti)
**Problem**: Kalau ERP dipakai multiple cabang/perusahaan dalam 1 instance.

**Mitigasi** (nanti, bukan sekarang):
- Global scope: `TenantScope` di Model → `where('company_id', auth()->user()->company_id)`
- Middleware: set tenant context
- Database: single DB + scope, atau database terpisah per tenant

## Operational Excellence

### 1. Error Handling Strategy

**Lapisan Error:**
```
Livewire → catch ValidationException → flash error + re-render
Service  → throw DomainException / ValidationException → rollback transaksi
Model    → EloquentException → Service catch + throw DomainException
Global   → Handler → render error page / JSON response
```

**Aturan:**
| Jenis Error | Response | Logging |
|---|---|---|
| Validasi form | Flash message error di form | Tidak perlu |
| Business logic (saldo tidak cukup) | Flash message + field highlight | `Log::info` |
| Database error | Flash message "Terjadi kesalahan sistem" | `Log::error` + trace |
| Not found (404) | Halaman 404 kustom | Tidak perlu |
| Forbidden (403) | Halaman 403 kustom | `Log::warning` |
| Server error (500) | Halaman 500 kustom | `Log::error` + trace |

**Polanya:**
```php
// Service layer — throw exception
class PegawaiService {
    public function create(array $data): Pegawai
    {
        DB::beginTransaction();
        try {
            $pegawai = Pegawai::create($data);
            ActivityLogger::log('created', $pegawai);
            DB::commit();
            return $pegawai;
        } catch (\Exception $e) {
            DB::rollBack();
            Log::error('Gagal create pegawai', [
                'data' => $data, 'error' => $e->getMessage(), 'trace' => $e->getTraceAsString()
            ]);
            throw new DomainException('Gagal menyimpan data pegawai: ' . $e->getMessage());
        }
    }
}
```

```php
// Livewire — catch exception, kasih feedback ke user
class PegawaiCreate extends Component {
    public function save()
    {
        $this->validate();
        try {
            app(PegawaiService::class)->create($this->all());
            session()->flash('success', 'Pegawai berhasil ditambahkan');
            return $this->redirect('/internal/hris/pegawai');
        } catch (DomainException $e) {
            $this->addError('form', $e->getMessage());
        } catch (\Exception $e) {
            Log::error('Unhandled error: ' . $e->getMessage());
            $this->addError('form', 'Terjadi kesalahan sistem. Silakan coba lagi.');
        }
    }
}
```

### 2. Permission Caching

**Problem**: Menu + permission diquery setiap request. Untuk ERP dengan 50+ menu dan 10+ role, query jadi berat.

**Solusi**: Cache permission per role, bust saat ada perubahan.

```php
class SidebarService {
    public function getModules(int $roleId): Collection
    {
        return Cache::remember("permissions.modules.{$roleId}", 3600, function () use ($roleId) {
            return Module::whereHas('menus.permissions', fn($q) =>
                $q->where('role_id', $roleId)->where('can_read', true)
            )->where('is_active', true)->orderBy('order')->get();
        });
    }

    public function getMenus(int $moduleId, int $roleId): Collection
    {
        return Cache::remember("permissions.menus.{$moduleId}.{$roleId}", 3600, function () use ($moduleId, $roleId) {
            return Menu::where('module_id', $moduleId)
                ->whereHas('permissions', fn($q) =>
                    $q->where('role_id', $roleId)->where('can_read', true)
                )->where('is_active', true)->orderBy('order')->get();
        });
    }

    // Panggil saat permission diubah oleh Admin
    public static function bustCache(int $roleId): void
    {
        Cache::forget("permissions.modules.{$roleId}");
        // Hapus semua cache menus untuk role ini
        $moduleIds = Module::pluck('id');
        foreach ($moduleIds as $moduleId) {
            Cache::forget("permissions.menus.{$moduleId}.{$roleId}");
        }
    }
}
```

**Trigger bust cache**:
- Setiap kali Admin ubah permission → `SidebarService::bustCache($roleId)`
- Setiap kali role/menu ditambah/dihapus → bust cache semua role

### 3. Activity Log Database

**Problem**: Log aktivitas user di file log tidak mudah dicari dan tidak permanen.

**Solusi**: Tabel `activity_logs` untuk audit trail permanen.

**Tabel:**
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | FK → users.id | nullable (guest action) |
| event | string | created, updated, deleted, approved, rejected, login, logout |
| subject_type | string | model class (App\Models\Pegawai) |
| subject_id | integer | nullable |
| description | text | "User A membuat data pegawai B" |
| old_values | jsonb | nullable — data sebelum diubah |
| new_values | jsonb | nullable — data setelah diubah |
| ip_address | string | nullable |
| user_agent | string | nullable |
| created_at | timestamp | |

**Auto-log Trait:**
```php
trait LogsActivity {
    protected static function bootLogsActivity(): void
    {
        static::created(fn($model) => ActivityLogger::log('created', $model));
        static::updated(fn($model) => ActivityLogger::log('updated', $model,
            $model->getOriginal(), $model->getChanges()
        ));
        static::deleted(fn($model) => ActivityLogger::log('deleted', $model));
    }
}
```

**Service manual:**
```php
class ActivityLogger {
    public static function log(string $event, $subject, ?array $old = null, ?array $new = null): void
    {
        ActivityLog::create([
            'user_id'    => auth()->id(),
            'event'      => $event,
            'subject_type'=> get_class($subject),
            'subject_id'  => $subject->id ?? null,
            'description' => self::generateDescription($event, $subject),
            'old_values'  => $old ? json_encode($old) : null,
            'new_values'  => $new ? json_encode($new) : null,
            'ip_address'  => request()->ip(),
            'user_agent'  => request()->userAgent(),
        ]);
    }
}
```

### 4. Testing Strategy

**Tingkat Test:**
| Layer | Tools | Coverage Target |
|---|---|---|
| Service | PHPUnit (Unit Test) | 90% — semua method Service harus di-test |
| Livewire | Livewire Test | 70% — render, validasi, event |
| Feature | PHPUnit (Feature Test) | Auth flow (login, register, throttling) |
| Browser | Laravel Dusk (optional) | Landing page, navigasi critical |

**Aturan:**
- Setiap Service Wajib punya test
- Test pakai **SQLite in-memory** untuk kecepatan
- Factory untuk data dummy
- Test dipisah per modul

```php
tests/
├── Unit/
│   └── Services/
│       └── Auth/
│           ├── LoginServiceTest.php
│           └── RegisterServiceTest.php
├── Feature/
│   ├── Auth/
│   │   ├── LoginTest.php
│   │   └── RegisterTest.php
│   └── Admin/
│       ├── RoleTest.php
│       └── MenuTest.php
└── Livewire/
    └── Auth/
        ├── LoginComponentTest.php
        └── RegisterComponentTest.php
```

**Test Database:**
```xml
<!-- phpunit.xml -->
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

**Perintah:**
```bash
php artisan make:test Services/Auth/LoginServiceTest --unit
php artisan make:test Feature/Auth/LoginTest
vendor/bin/phpunit
```

## Struktur File

```
project/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── LoginController.php
│   │   │   │   └── LogoutController.php
│   │   │   ├── Internal/
│   │   │   │   └── DashboardController.php
│   │   │   └── Eksternal/
│   │   │       └── DashboardController.php
│   │   ├── Middleware/
│   │   │   ├── CheckInternal.php
│   │   │   ├── CheckEksternal.php
│   │   │   └── CheckSuperAdmin.php
│   │   └── Requests/
│   │       └── Auth/
│   │           └── RegisterRequest.php
│   │
│   ├── Livewire/                 ← CONTROLLER LAYER
│   │   ├── Auth/
│   │   │   ├── Login.php
│   │   │   └── Register.php
│   │   ├── Internal/
│   │   │   ├── Dashboard.php
│   │   │   ├── Sidebar.php
│   │   │   └── ModuleGrid.php
│   │   ├── Eksternal/
│   │   │   ├── Dashboard.php
│   │   │   ├── Sidebar.php
│   │   │   └── ModuleGrid.php
│   │   ├── Admin/
│   │   │   ├── Dashboard.php
│   │   │   ├── Sidebar.php
│   │   │   ├── UserApproval.php
│   │   │   ├── ModuleManager.php
│   │   │   ├── MenuManager.php
│   │   │   ├── RoleManager.php
│   │   │   └── PermissionManager.php
│   │   └── Modules/              ← PER MODUL
│   │       ├── Hris/
│   │       │   ├── PegawaiIndex.php
│   │       │   ├── PegawaiCreate.php
│   │       │   └── PegawaiEdit.php
│   │       ├── Marketing/
│   │       ├── Finance/
│   │       └── Eproc/
│   │
│   ├── Services/                 ← BUSINESS LOGIC
│   │   ├── Auth/
│   │   │   ├── LoginService.php
│   │   │   └── RegisterService.php
│   │   ├── Admin/
│   │   │   ├── UserApprovalService.php
│   │   │   ├── RoleService.php
│   │   │   └── MenuService.php
│   │   ├── SidebarService.php
│   │   └── Modules/              ← PER MODUL
│   │       ├── Hris/
│   │       │   ├── PegawaiService.php
│   │       │   └── AbsensiService.php
│   │       ├── Marketing/
│   │       ├── Finance/
│   │       └── Eproc/
│   │
│   ├── Models/                   ← DATA LAYER
│   │   ├── User.php
│   │   ├── Role.php
│   │   ├── Module.php
│   │   ├── Menu.php
│   │   ├── Permission.php
│   │   └── Modules/              ← PER MODUL
│   │       ├── Hris/
│   │       │   ├── Pegawai.php
│   │       │   └── Absensi.php
│   │       ├── Marketing/
│   │       └── Finance/
│   │
│   └── Traits/
│       ├── AuditTrail.php
│       └── ActivityLogger.php
│
├── resources/views/              ← VIEW LAYER
│   ├── components/
│   │   ├── layout-internal.blade.php
│   │   ├── layout-eksternal.blade.php
│   │   ├── layout-admin.blade.php
│   │   └── layout-landing.blade.php
│   ├── landing/
│   │   ├── partials/
│   │   │   ├── hero.blade.php
│   │   │   ├── features.blade.php
│   │   │   ├── about.blade.php
│   │   │   ├── contact.blade.php
│   │   │   └── footer.blade.php
│   │   └── index.blade.php
│   ├── auth/
│   │   ├── login.blade.php
│   │   └── register.blade.php
│   ├── admin/
│   │   ├── dashboard.blade.php
│   │   ├── sidebar.blade.php
│   │   ├── user-approval.blade.php
│   │   ├── module-manager.blade.php
│   │   ├── menu-manager.blade.php
│   │   ├── role-manager.blade.php
│   │   └── permission-manager.blade.php
│   ├── internal/
│   │   ├── dashboard.blade.php
│   │   ├── sidebar.blade.php
│   │   └── module-grid.blade.php
│   ├── eksternal/
│   │   ├── dashboard.blade.php
│   │   ├── sidebar.blade.php
│   │   └── module-grid.blade.php
│   └── modules/                  ← PER MODUL
│       ├── hris/
│       │   ├── pegawai/
│       │   │   ├── index.blade.php
│       │   │   ├── create.blade.php
│       │   │   └── edit.blade.php
│       │   └── absensi/
│       │       └── index.blade.php
│       ├── marketing/
│       └── finance/
│
├── routes/
│   ├── web.php
│   ├── admin.php
│   ├── internal.php
│   └── eksternal.php
│
├── database/
│   ├── migrations/
│   ├── seeders/
│   │   ├── RoleSeeder.php
│   │   ├── ModuleSeeder.php
│   │   ├── MenuSeeder.php
│   │   ├── UserSeeder.php
│   │   └── PermissionSeeder.php
│   └── factories/
│
├── package.json
└── composer.json
```
