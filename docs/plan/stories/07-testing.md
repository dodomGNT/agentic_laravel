# Story: Testing Setup & Strategy

## Description
Setup PHPUnit, database testing (SQLite in-memory), factories, dan test dasar untuk Service & Auth.

## Acceptance Criteria
### Setup
- [ ] Konfigurasi `phpunit.xml`: DB_CONNECTION=sqlite, DB_DATABASE=:memory:
- [ ] Folder `tests/Unit/Services/` + `tests/Feature/Auth/` + `tests/Livewire/Auth/`
- [ ] Factory: UserFactory, RoleFactory, ModuleFactory, MenuFactory

### Service Tests
- [ ] `LoginServiceTest` — test login sukses, gagal, throttling (unit test)
- [ ] `RegisterServiceTest` — test register sukses, duplicate email, weak password
- [ ] `SidebarServiceTest` — test getModules, getMenus berdasarkan role

### Feature Tests
- [ ] `LoginTest` — test halaman login render, form submit sukses, form submit gagal
- [ ] `RegisterTest` — test register flow, redirect setelah register
- [ ] `ThrottlingTest` — test 5 failed attempts → block 1 menit

### Livewire Tests
- [ ] `LoginComponentTest` — test render, validasi, event dispatch
- [ ] `RegisterComponentTest` — test render, password strength, submit

### Perintah
- [ ] `vendor/bin/phpunit` — semua test jalan
- [ ] Test bisa jalan tanpa koneksi PostgreSQL (pakai SQLite memory)

## Technical Notes
- Install: PHPUnit sudah include di Laravel
- Factory: `php artisan make:factory UserFactory`
- Test: `php artisan make:test Services/Auth/LoginServiceTest --unit`
- Pastikan Model::preventLazyLoading() di-nonaktifkan saat test (false)

## Dependencies
- Story 01 (Setup)
