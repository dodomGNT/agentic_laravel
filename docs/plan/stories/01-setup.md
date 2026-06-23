# Story: Setup Laravel + Livewire + Database + Models

## Description
Instalasi Laravel, konfigurasi PostgreSQL, install Livewire 3, migrasi tabel, model dengan soft delete & audit trail, seeder.

## Acceptance Criteria
- [ ] Laravel 12 terinstall dengan PostgreSQL
- [ ] Livewire 3 terinstall via Composer
- [ ] Blade Heroicons: `composer require blade-ui-kit/blade-heroicons`
- [ ] Composer packages:
  - `composer require barryvdh/laravel-dompdf` (PDF export)
  - `composer require maatwebsite/laravel-excel` (Excel import/export)
- [ ] Frontend packages via npm:
  - `npm i -D daisyui@latest tailwindcss @tailwindcss/vite`
  - `npm i aos flatpickr filepond filepond-plugin-image-preview`
  - `npm i tom-select trix`
  - `npm i leaflet leaflet-draw` (peta + poligon)
  - `npm i apexcharts` (chart interaktif)
- [ ] Konfigurasi Tailwind v4 + daisyUI di `vite.config.js`
- [ ] AOS init + flatpickr + Tom Select + FilePond register di `resources/js/app.js`
- [ ] Import CSS: daisyUI, flatpickr, filepond, tom-select di `resources/css/app.css`
- [ ] **Laravel strict mode**: `Model::preventLazyLoading()` di `AppServiceProvider`
- [ ] **Queue table**: `php artisan queue:table` + migrate
- [ ] **Activity log table**: migrasi `activity_logs` (user_id, event, subject_type, subject_id, description, old_values jsonb, new_values jsonb, ip_address, user_agent)
- [ ] **Traits**: `LogsActivity` trait untuk auto-log, `ActivityLogger` service
- [ ] **Cache & optimize**: `php artisan optimize` untuk production
- [ ] Migrasi tabel: modules, roles, users, menus, permissions
- [ ] Schema:

**modules** — id, name, slug(unique), icon, description(nullable), is_eksternal(bool), order(int), is_active(bool), timestamps

**roles** — id, name, is_eksternal(bool), timestamps

**users** — id, name, email(unique), password(hashed), role_id(FK nullable), is_approved(bool), is_active(bool), is_eksternal(bool), created_by(FK nullable), updated_by(FK nullable), softDeletes, timestamps

**menus** — id, module_id(FK), name, route, icon, parent_id(FK nullable), order(int), is_active(bool), timestamps

**permissions** — id, role_id(FK), menu_id(FK), can_create(bool), can_read(bool), can_update(bool), can_delete(bool), can_approve(bool), unique(role_id, menu_id), timestamps

- [ ] Model: User(SoftDeletes, created_by, updated_by, belongsTo Role), Role, Module(hasMany Menu), Menu(belongsTo Module, belongsTo parent, hasMany children), Permission(belongsTo Role, belongsTo Menu)
- [ ] Soft Delete Trait di model User
- [ ] created_by / updated_by via model boot() atau trait
- [ ] Seeder: 4 role (superadmin, admin, user, vendor)
- [ ] Seeder: 4 module (HRIS, Marketing, Finance, Eproc)
- [ ] Seeder: menu default per module (2-3 menu tiap module)
- [ ] Seeder: 1 user superadmin
- [ ] Seeder: permission untuk role superadmin (semua akses)

## Technical Notes
- Livewire 3: `composer require livewire/livewire`
- Blade Heroicons: `composer require blade-ui-kit/blade-heroicons`
- SoftDeletes: `use Illuminate\Database\Eloquent\SoftDeletes;`
- Pastikan urutan migrasi: modules → roles → users → menus → permissions
- `php artisan make:seeder RoleSeeder ModuleSeeder MenuSeeder UserSeeder PermissionSeeder`
- Konfigurasi Tailwind v4: `@import "tailwindcss";` + `@plugin "daisyui";` di `app.css`
- Konfigurasi Vite: `tailwindcss()` plugin di `vite.config.js`
- AOS: `import AOS from 'aos'; AOS.init();` di `app.js`
- flatpickr: `import flatpickr from 'flatpickr';` + `import 'flatpickr/dist/flatpickr.min.css';`
- FilePond: `import * as FilePond from 'filepond';` + `import FilePondPluginImagePreview from 'filepond-plugin-image-preview';`
- Tom Select: `import TomSelect from 'tom-select';` + `import 'tom-select/dist/css/tom-select.css';`
- Semua library diinisialisasi via Alpine.js `x-init` di Blade component
- Leaflet: `import L from 'leaflet';` + `import 'leaflet/dist/leaflet.css';` + `import 'leaflet-draw';`
- ApexCharts: `import ApexCharts from 'apexcharts';` + `import 'apexcharts/dist/apexcharts.css';`
- domPDF: konfigurasi di `config/dompdf.php`, font support via `loadHTML()`
- **Migration strategy**: 1 migration per modul (contoh: `create_hris_tables.php`) untuk mengurangi jumlah file
- **Queue**: `QUEUE_CONNECTION=database` di `.env`, jalankan `php artisan queue:work` untuk development
- **Strict mode**: `AppServiceProvider::boot() { Model::preventLazyLoading(); }`

## Dependencies
- None
