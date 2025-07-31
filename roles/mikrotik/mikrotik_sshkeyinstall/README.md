# mikrotik_sshkeyinstall

An Ansible role to install SSH public keys for users on MikroTik RouterOS devices.

---

## 📄 Description

This role automates the installation of SSH public keys for local users on MikroTik RouterOS devices (tested on RouterOS 7.x). It performs key and user validation, uploads key files via SSH, and imports them into RouterOS configuration.

Modular task design allows for improved control and flexibility.

Features include:

- Validation of SSH public key files (existence, syntax, supported type)
- Verification that users exist on MikroTik before applying changes
- Optional removal of existing SSH keys before importing
- Graceful handling of missing or invalid inputs

---

## ✅ Requirements

- **MikroTik RouterOS** (tested on v7.19.3)
- **Ansible** 2.9+  
  Required collections:
  - `community.routeros`
  - `ansible.netcommon`

The role uses SSH to interact with MikroTik using the `network_cli` connection plugin and the `routeros` network OS.

---

## 📝 Role Variables

| Variable                   | Description                                                                                   | Default                        |
|---------------------------|-----------------------------------------------------------------------------------------------|--------------------------------|
| `mikrotik_ssh_users`      | List of users and their public key files                                                      | `[]`                           |
| `pubkeys_path`            | Base directory path where `.pub` key files are stored                                         | `{{ role_path }}/files`        |
| `verify_users_exist`      | Whether to verify that each user exists on MikroTik                                           | `true`                         |
| `fail_on_invalid_user`    | If `true`, role fails when a user does not exist on MikroTik                                  | `false`                        |
| `verify_pubkeys`          | Whether to check public key files for existence and format validity                           | `true`                         |
| `fail_on_invalid_pubkey`  | If `true`, role fails on missing or invalid public key files                                  | `false`                        |
| `allow_flushing_pubkeys`  | If `true`, removes existing SSH keys for users with keys provided before importing new ones   | `false`                        |

### 🔐 `mikrotik_ssh_users` Structure

```yaml
mikrotik_ssh_users:
  - name: John
    pubkey_file: John-ed25519.pub
  - name: Tom
    pubkey_file: Tom-rsa.pub
```

---

## 📁 Role Structure (Modular Tasks)

The role is split into modular task files for clarity and maintainability:

- `verify_pubkeys.yml`: Validates existence and format of public key files
- `fetch_mikrotik_users.yml`: Fetches current user list from MikroTik
- `verify_mikrotik_users.yml`: Verifies requested users against MikroTik
- `setup_upload_list.yml`: Builds final list of valid users+keys
- `flush_ssh_pubkeys.yml`: Removes existing keys (if enabled)
- `upload_pubkeys.yml`: Uploads `.pub` files via SSH
- `import_pubkeys.yml`: Imports uploaded keys into RouterOS

---

## 🔒 Key Flushing Behavior

If `allow_flushing_pubkeys` is set to `true`, the role will remove **all existing SSH keys** for users listed in `mikrotik_ssh_users`, before importing the new ones.

This is necessary because MikroTik does not expose current key contents, making updates idempotent only if keys are flushed first.

If disabled, **imported keys will accumulate**, possibly resulting in duplicates.

---

## 🧪 Key and User Validation

When enabled, the role checks:

- If each user defined in `mikrotik_ssh_users` exists on the device
- If each `.pub` file exists and is a valid supported type:
  - Allowed types: `ssh-rsa`, `ssh-ed25519`

Validation is controlled using the following flags:

- `verify_users_exist`
- `fail_on_invalid_user`
- `verify_pubkeys`
- `fail_on_invalid_pubkey`

---

## 🔧 Example Inventory

```ini
[mikrotik]
router1 ansible_host=192.0.2.1

[mikrotik:vars]
ansible_port=22
ansible_user=ansible_user
ansible_connection=network_cli
ansible_network_os=routeros
```

---

## ▶️ Example Playbook

```yaml
- name: Install SSH keys on MikroTik
  hosts: mikrotik
  gather_facts: false

  vars:
    allow_flushing_pubkeys: true
    fail_on_invalid_user: true
    fail_on_invalid_pubkey: true

    mikrotik_ssh_users:
      - name: John
        pubkey_file: John-rsa.pub
      - name: John
        pubkey_file: John-ed25519.pub
      - name: Tom
        pubkey_file: Tom-ed25519.pub

  roles:
    - role: mikrotik_sshkeyinstall
```

> 📂 Make sure all `.pub` files are located in the role's `files/` directory (e.g. `roles/mikrotik_sshkeyinstall/files/John-ed25519.pub`).

---

## 🧾 Notes

- Uses the `community.routeros.command` module for CLI commands
- Uploads key files using `ansible.netcommon.net_put`
- Task execution is delegated to `localhost` when handling `.pub` file validation

---

## 🪪 License

This role is licensed under the **MIT-0 License**, allowing free use, modification, and distribution with no restrictions.

---

## 🧑‍💻 Author Information

**Paweł Wołochow**  
Email: [wolochowp@gmail.com](mailto:wolochowp@gmail.com)  
GitHub: [wolochowp](https://github.com/wolochowp)