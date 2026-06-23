# Story: User Approval + Dashboard Layouts

## Description
Super Admin dapat melihat daftar user pending, approve, dan assign role. Halaman dashboard internal dan eksternal dengan layout masing-masing.

## Acceptance Criteria
- [ ] Halaman daftar user pending (hanya untuk superadmin)
- [ ] Approve user → is_approved = true
- [ ] Assign role ke user saat approve
- [ ] Halaman `/internal/dashboard` dengan layout internal sidebar
- [ ] Halaman `/eksternal/dashboard` dengan layout eksternal sidebar
- [ ] Layout internal: sidebar kiri menu internal, navbar, konten
- [ ] Layout eksternal: sidebar kiri menu eksternal, navbar, konten
- [ ] Landing page `/` untuk guest

## Technical Notes
- Blade layout: `layouts/internal.blade.php`, `layouts/eksternal.blade.php`
- Sidebar partial: `components/internal-sidebar.blade.php`, `components/eksternal-sidebar.blade.php`

## Dependencies
- Story 03 (Menu & Permission)
