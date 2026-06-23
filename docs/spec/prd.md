# PRD: Admin Panel — Sistem Informasi Perusahaan (ERP)

## Problem Statement
Perusahaan membutuhkan sistem informasi terpusat yang mencakup modul HRIS, Marketing, Finance, dan Eproc. Sistem harus membedakan akses antara pegawai internal dan vendor eksternal, dengan keamanan berbasis role & permission.

## Goals
- Membangun web aplikasi ERP modular dengan Laravel
- Memisahkan akses internal dan eksternal via routing berbeda
- Auth sederhana dengan registrasi + approval Super Admin
- Sidebar menu dinamis berdasarkan role & permission

## Tipe User

| Tipe | Login? | Akses |
|---|---|---|
| Guest | Tidak | Landing page saja |
| Eksternal | Ya | `/eksternal/dashboard` + modul Eproc |
| Internal | Ya | `/internal/dashboard` + seluruh modul internal |

## Modul

### 1. Internal
1.1. HRIS — data pegawai, absensi, penggajian
1.2. Marketing — leads, campaign, report
1.3. Finance — accounting, budgeting, billing
1.4. Eproc — procurement (pengadaan)

### 2. Eksternal
2.1. Eproc — vendor portal untuk pengadaan

## Auth & Routing
- Route prefix: `/internal/*` dan `/eksternal/*`
- Registrasi user → pending approval
- Super Admin menentukan role user setelah registrasi
- User eksternal tidak bisa akses `/internal/*`
- User internal tidak bisa akses `/eksternal/*`

## Role

| Role | isEksternal | Keterangan |
|---|---|---|
| superadmin | false | Akses penuh internal |
| admin | false | Kelola internal |
| user | false | Pegawai biasa |
| vendor | true | Pihak luar |

## Menu & Permission
- Menu disusun secara hierarki (parent-child)
- Setiap menu ditandai: internal atau eksternal
- Permission = relasi antara Menu × Role
- Sidebar menampilkan menu sesuai permission user

## Out of Scope
- Modul HRIS, Marketing, Finance, Eproc hanya sebagai placeholder (belum implementasi fitur detail)
- Notifikasi real-time
- API eksternal integration

## Success Metrics
- Landing page bisa diakses guest
- Registrasi → approval → login sukses
- Sidebar hanya menampilkan menu sesuai role
- Internal tidak bisa akses `/eksternal/*` dan sebaliknya
