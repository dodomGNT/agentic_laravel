# Story: Auth System (Livewire) + Middleware + Security

## Description
Registrasi dengan verifikasi email, login dengan throttling + single session + IP detection, middleware, route grouping, audit trail.

## Acceptance Criteria

### Database
- [ ] Migrasi: tambah kolom `email_verified_at` (timestamp, nullable) ke users
- [ ] Migrasi: tambah kolom `last_login_at` (timestamp, nullable)
- [ ] Migrasi: tambah kolom `last_login_ip` (string, nullable)
- [ ] Migrasi: tambah kolom `session_id` (string, nullable)

### Desain Halaman Auth (Login & Register)
Kedua halaman menggunakan layout **split-screen** dengan visual modern:

**Layout Split-Screen (Desktop):**
- Kiri (50%): branding area — gradient/pattern/ilustrasi atau testimonial carousel
  - Logo perusahaan besar
  - Quote/caption brand
  - Atau ilustrasi vector abstrak dengan animasi
  - Atau carousel testimoni user (auto-slide dengan Alpine.js)
- Kanan (50%): form card
  - Glassmorphism card (bg-white/70 backdrop-blur) atau daisyUI card
  - Logo kecil di atas form
  - Judul: "Selamat Datang" / "Buat Akun"
  - Subtitle kecil
  - Form input dengan daisyUI styling
  - Tombol CTA dengan loading state
  - Link ke halaman sebaliknya (Register ↔ Login)

**Mobile (< 768px):**
- Fullscreen form saja (split dihilangkan)
- Background gradient sederhana
- Form di tengah layar

**Elemen Visual:**
- [ ] Split-screen layout (kiri brand, kanan form)
- [ ] Glassmorphism card (backdrop-blur + semi-transparan) atau daisyUI card shadow-xl
- [ ] Logo perusahaan di form
- [ ] Animasi fade-in saat halaman dimuat (Alpine.js x-transition)
- [ ] Gradient background di sisi brand
- [ ] Testimonial carousel di sisi brand (Alpine.js auto-slide)
- [ ] Social proof text: "Digunakan oleh 500+ perusahaan"
- [ ] Error/success notification: toast modern di pojok kanan atas (daisyUI alert + Alpine.js auto-dismiss 5 detik)
- [ ] Responsive: mobile stack jadi satu kolom
- [ ] Font scale: judul large, label normal, helper text small
- [ ] Password strength indicator muncul di bawah field password (bukan toast)

### Registrasi (Livewire)
- [ ] Livewire `Auth\Register`: form name, email, password, password_confirmation
- [ ] Password strength indicator real-time dengan Alpine.js
- [ ] Validasi: email unique, password min 8 + huruf + angka
- [ ] **Loading**: button submit disabled + spinner "Mendaftarkan..." selama proses
- [ ] Post-registrasi: kirim email verifikasi
- [ ] Di development: tampilkan link verifikasi di flash message (skip mail)
- [ ] Redirect ke login: "Cek email untuk verifikasi akun Anda"

### Verifikasi Email
- [ ] Route `/verify-email/{token}` — set email_verified_at = now
- [ ] Redirect ke login: "Email berhasil diverifikasi. Tunggu persetujuan admin."
- [ ] Token: hash dari email + timestamp, simpan di cache (expire 24 jam)

### Login (Livewire)
- [ ] Livewire `Auth\Login`: form email + password
- [ ] Login throttling: max 5 attempts per menit per email+IP
- [ ] Jika throttled: pesan error inline "Terlalu banyak percobaan. Coba lagi {detik} detik."
- [ ] **Loading**: button submit disabled + spinner "Memproses..." selama proses login
- [ ] Cek email_verified_at — belum: "Verifikasi email Anda terlebih dahulu"
- [ ] Cek is_approved — belum: "Akun Anda belum diverifikasi admin"
- [ ] Cek is_active — false: "Akun Anda telah dinonaktifkan"
- [ ] Single session: jika user sudah login di session lain, invalidate session lama
- [ ] Update: last_login_at, last_login_ip, session_id
- [ ] IP anomaly: cek apakah IP ini baru untuk user ini, jika ya → log warning
- [ ] Audit trail: log failed attempt (email, IP, timestamp) — `Log::warning`
- [ ] Audit trail: log successful login (user_id, IP, timestamp) — `Log::info`
- [ ] Redirect sukses ke /internal/dashboard atau /eksternal/dashboard

### Dashboard (Last Login Info)
- [ ] Tampilkan di dashboard: "Terakhir login: {tanggal} dari IP {ip}"
- [ ] Jika login dari IP baru: notifikasi "Login terdeteksi dari IP baru: {ip}"

### Middleware
- [ ] CheckInternal: user login + role->is_eksternal == false
- [ ] CheckEksternal: user login + role->is_eksternal == true
- [ ] CheckSuperAdmin: user login + role->name == superadmin
- [ ] Jika tidak punya akses → abort(403)

### Routing
- [ ] /register → Livewire Auth\Register
- [ ] /login → Livewire Auth\Login
- [ ] /logout → Controller logout + update session_id = null + log audit
- [ ] /verify-email/{token} → Controller verifikasi
- [ ] /internal/{module?}/{page?} → middleware web + auth + CheckInternal
- [ ] /eksternal/{module?}/{page?} → middleware web + auth + CheckEksternal
- [ ] /admin/{section?}/{action?} → middleware web + auth + CheckSuperAdmin

### Logout
- [ ] Clear session + set session_id = null di users
- [ ] Log audit: logout (user_id, timestamp)

## Technical Notes
- Throttling: `Illuminate\Cache\RateLimiter` — key: `'login:' . strtolower($email) . '|' . request()->ip()`
- Single session: di middleware, `if (auth()->user()->session_id !== session()->getId()) { auth()->logout(); ... }`
- Password strength: Alpine.js — `x-data="{ password: '', strength: 0 }"` dengan watcher
- IP history: simpan di log saja, atau cache dengan key `ip_history:{user_id}` (last 5 IPs)
- Email verification token: `sha1($email . now()->timestamp)` disimpan di cache 24 jam
- Notification: flash message + Tailwind CSS alert component

## Dependencies
- Story 01 (Setup)
