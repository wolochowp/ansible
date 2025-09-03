# Ansible Roles

A curated collection of reusable and well-structured **Ansible roles** designed for automation of real-world infrastructure tasks — beginning with MikroTik RouterOS backup management and SSH key provisioning.

---

## 📦 Roles Included

### `mikrotik_backup`
Backup role for MikroTik RouterOS devices
📁 Location: `roles/mikrotik_backup`

- Supports both `.backup` and `.rsc` MikroTik backup formats.
- Works with:
  - **Direct SFTP transfer** (from MikroTik using `/tool fetch upload=yes`)
  - **Indirect transfer** (via Ansible controller using `sftp` or `sshpass`)
- Handles both SSH key and password-based authentication for SFTP.
- Automatically cleans up temporary files on the MikroTik and Ansible controller.
- Suitable for environments with or without outbound MikroTik access.

---

### `mikrotik_sshkeyinstall`
Role to install SSH public keys for MikroTik users
📁 Location: `roles/mikrotik_sshkeyinstall`

- Installs `.pub` SSH keys (`ssh-rsa` and `ssh-ed25519`) on MikroTik users.
- Optional verification of:
  - Whether users exist on the router (`verify_users_exist`)
  - Whether public key files exist in the role (`verify_pubkeys`)
  - Whether keys are valid/compatible (`fail_on_invalid_pubkey`)
- Ensures idempotent deployment using an optional **flush mode** (`allow_flushing_pubkeys`):
  - MikroTik **cannot show** which public keys are already installed.
  - To avoid **duplicate keys** on every run, the role **flushes** existing keys for users whose keys are managed by Ansible.
  - Flushing **only affects users** that have public keys defined in the play — other users and their keys are untouched.

> ⚠️ By default, `allow_flushing_pubkeys` is **disabled** to avoid unintentional key removal. Enable it only when managing a user's keys fully via Ansible.

---

### `hashivault_deploy`
Role to install, configure, and bootstrap a **HashiCorp Vault cluster**
📁 Location: `roles/hashivault_deploy`

- Installs Vault from official releases and configures systemd service for Linux hosts.
- Supports multiple storage backends: `raft` (recommended) and `file`.
- Supports Shamir and cloud KMS-based auto-unseal (`awskms`, `azurekeyvault`, `gcpckms`, `hsm`, `transit`).
- TLS and mTLS support with wildcard or per-node certificates.
- Systemd hardening options for secure Vault deployment.
- Dynamic cluster join via `retry_join` or manual Raft join.
- Vault token and recovery key management with Shamir key shares.
- Fully customizable via role variables, including Vault ports, bind addresses, cluster names, and telemetry.
- Idempotent and production-ready deployment with optional bootstrapping of Vault root token.

> ⚠️ Sensitive data (root tokens, unseal keys) is handled securely with optional `no_log` flags. Ensure Vault nodes are reachable from the Ansible controller.

---
## 🚀 Getting Started

Each role includes a `README.md` with detailed usage, examples, variables, and requirements. Just import them into your playbook and inventory, and you're ready to automate MikroTik like a pro.
