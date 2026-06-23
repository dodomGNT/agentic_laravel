# Story: Landing Page — Desain Modern & Atraktif

## Description
Landing page publik dengan desain paling modern dan atraktif. Single page website dengan Hero, Features, About, Contact, Footer.

## Acceptance Criteria

### Setup Frontend
- [ ] Install daisyUI v5 via npm (`npm i -D daisyui@latest`)
- [ ] Konfigurasi Tailwind v4 + daisyUI di `vite.config.js`
- [ ] Install AOS (`npm i aos`) untuk scroll animation
- [ ] Install Blade Heroicons (`composer require blade-ui-kit/blade-heroicons`)
- [ ] Setup Google Fonts Inter di layout
- [ ] Setup theme dark/light switching via daisyUI

### Navbar (Sticky)
- [ ] Logo kiri, navigasi tengah (Beranda, Fitur, Tentang, Kontak), CTA "Login" kanan
- [ ] Sticky di atas, background blur/transparan saat di-scroll
- [ ] Mobile responsive: hamburger menu
- [ ] Smooth scroll ke section (href="#features" dll)

### Hero Section
- [ ] Headline besar: "Sistem Informasi Perusahaan Terpadu"
- [ ] Subtext: deskripsi singkat ERP
- [ ] CTA button: "Mulai Sekarang" → scroll ke registrasi / fitur
- [ ] Background: gradient modern atau ilustrasi/pattern
- [ ] AOS animation: fade-up

### Features Section (AOS)
- [ ] Grid 4 card fitur utama: HRIS, Marketing, Finance, Eproc
- [ ] Setiap card: icon Heroicons + judul + deskripsi + efek hover
- [ ] AOS: fade-up bergantian (stagger)

### About Us Section
- [ ] Company story / visi misi
- [ ] Stats counter animasi: 500+ Users · 4 Modul · 99.9% Uptime · 24/7 Support
- [ ] AOS: fade-left untuk teks, fade-right untuk gambar/ilustrasi

### Contact Section
- [ ] Card modern dengan form: nama, email, pesan
- [ ] Atau informasi kontak (alamat, email, telepon)
- [ ] **Loading**: button submit spinner "Mengirim..." saat form di-submit
- [ ] AOS: zoom-in

### Footer
- [ ] 4 kolom: Produk, Perusahaan, Dukungan, Ikuti Kami
- [ ] Social media icons
- [ ] Copyright
- [ ] daisyUI footer component

### Responsive
- [ ] Mobile-first design
- [ ] Navbar jadi hamburger di mobile
- [ ] Grid jadi stack di mobile
- [ ] Font scaling responsive

### Animasi
- [ ] AOS on scroll untuk semua section
- [ ] Hover effects pada card dan button
- [ ] Smooth scroll navigation

## Technical Notes
- File: `resources/views/landing.blade.php`
- Section partials di `resources/views/landing/` (hero.blade.php, features.blade.php, dll)
- Layout terpisah dari admin (`resources/views/components/layout-landing.blade.php`)
- AOS init di `resources/js/app.js`: `import AOS from 'aos'; AOS.init();`

## Dependencies
- Story 01 (Setup Laravel) — untuk akses ke file frontend
