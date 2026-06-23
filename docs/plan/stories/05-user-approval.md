# Story: User Approval + Admin Dashboard

## Description
Super Admin approve user registrasi, assign role, kelola user.

## Acceptance Criteria
- [ ] Livewire `Admin\UserApproval`: table user pending (is_approved=false)
- [ ] Approve user → set is_approved=true, pilih role dari dropdown
- [ ] **Loading**: button "Approve" disabled + spinner "Menyetujui..."
- [ ] Tolak user → hapus user
- [ ] **Loading**: button "Tolak" disabled + spinner
- [ ] Livewire `Admin\Dashboard`: overview statistik (total user, pending, dll)
- [ ] **Loading**: card statistik skeleton pulse saat load
- [ ] Sidebar admin: menu Users, Roles, Menus, Permissions, Dashboard

## Dependencies
- Story 02 (Auth)
