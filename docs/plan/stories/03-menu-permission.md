# Story: Module + Menu + Permission System (Livewire CRUD)

## Description
CRUD Module, CRUD Menu (per module), CRUD Role, Permission assignment (grid role×menu).

## Acceptance Criteria
### Module
- [ ] Livewire `Admin\ModuleManager`: list, create, edit, delete module
- [ ] Field: name, slug (auto generated), icon, description, is_eksternal, order, is_active
- [ ] Module slug dipakai untuk routing: /internal/{slug}/{page?}
- [ ] Seeder: 4 module — HRIS, Marketing, Finance, Eproc
- [ ] **Loading**: button submit spinner "Menyimpan...", tabel skeleton saat load

### Menu
- [ ] Livewire `Admin\MenuManager`: list per module, create, edit, delete menu
- [ ] Field: module_id (FK), name, route, icon, parent_id (nullable), order, is_active
- [ ] Menu terkait dengan module via module_id
- [ ] Dropdown parent_id hanya menampilkan menu dalam module yang sama
- [ ] route = nama route untuk navigasi halaman
- [ ] **Loading**: button submit spinner, tabel skeleton

### Role
- [ ] Livewire `Admin\RoleManager`: list, create, edit, delete role
- [ ] Field: name, is_eksternal
- [ ] **Loading**: button submit spinner, tabel skeleton

### Permission
- [ ] Livewire `Admin\PermissionManager`: table Role × Menu dengan checkbox
- [ ] Permission actions: can_create, can_read, can_update, can_delete, can_approve
- [ ] Cascade delete: saat module/menu/role dihapus, permission terkait ikut terhapus
- [ ] **Loading**: saat menyimpan permission, button "Simpan" disabled + spinner

### SidebarService
- [ ] `getModules($roleId)` → module yang memiliki menu accessible oleh role user
- [ ] `getMenus($moduleId, $roleId)` → menu dalam module yang di-allow role
- [ ] Filter: is_eksternal menyesuaikan tipe user

## Dependencies
- Story 02 (Auth)
