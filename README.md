# Ansible Roles by Paweł Wołochow

A curated collection of reusable and well-structured **Ansible roles** designed for automation of real-world infrastructure tasks — beginning with MikroTik RouterOS backup management.

---

## 📦 Roles Included

### `mikrotik_backup`
Backup role for MikroTik RouterOS devices  
📁 Location: `roles/mikrotik/mikrotik_backup
- Supports both `.backup`, `.rsc` mikrotik backup types.
- Works with **direct SFTP transfer** from MikroTik or via **indirect transfer** through the Ansible controller.
- Handles SFTP uploads using SSH keys or password-based auth (via `sshpass`).
- Automatically cleans up temporary files on controller and router.
