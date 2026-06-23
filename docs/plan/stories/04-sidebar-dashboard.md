# Story: Sidebar Navigation + Module Grid + Dashboard

## Description
Livewire sidebar component dengan state level 1/2, module grid, dashboard page, layout internal & eksternal.

## Acceptance Criteria
### Layout
- [ ] Component `layout-internal`: sidebar kiri + konten kanan
- [ ] Component `layout-eksternal`: sidebar kiri + konten kanan
- [ ] Component `layout-admin`: sidebar kiri + konten kanan

### Sidebar (Livewire)
- [ ] `Internal\Sidebar` component
- [ ] State: $level (1|2), $currentModule, $currentPage
- [ ] Level 1: tampilkan daftar module (parent menu) sesuai role user
- [ ] Level 2: tampilkan sub-menu module + tombol "← Kembali"
- [ ] `selectModule($module)` → set level=2, dispatch event 'moduleSelected'
- [ ] `selectPage($page)` → dispatch event 'pageSelected'
- [ ] `back()` → set level=1, dispatch event 'backToModules'
- [ ] **Loading**: saat klik module/page, konten area tampilkan skeleton loading (3 baris shimmer), sidebar tetap aktif

### ModuleGrid (Livewire)
- [ ] `Internal\ModuleGrid` component
- [ ] Render grid/list sub-menu berdasarkan module yang dipilih
- [ ] Setiap item bisa diklik → dispatch 'pageSelected'
- [ ] **Loading**: card grid menampilkan skeleton pulse saat load data
- [ ] `Eksternal\ModuleGrid` (sama, filtered untuk eksternal)

### Dashboard (Full-page Livewire)
- [ ] `Internal\Dashboard` — #[Layout('components.layout-internal')]
- [ ] mount($module, $page) — inisialisasi state dari URL
- [ ] Render sidebar + konten sesuai state
- [ ] Listen event dari sidebar → update konten
- [ ] **Loading**: konten area overlay spinner dengan `wire:loading` wrapper
- [ ] `Eksternal\Dashboard` — pattern serupa

### Navigation Flow
```
State: level=1, module=null, page=null → sidebar: module list, konten: dashboard

klik module "HRIS":
  → sidebar: level=2 module=hris, konten: ModuleGrid(hris)

klik sub-menu "Pegawai":
  → sidebar: level=2 module=hris page=pegawai, konten: PlaceholderPage

klik "Kembali":
  → sidebar: level=1, konten: dashboard
```

## Dependencies
- Story 03 (Menu & Permission)
