# mikrotik_sshkeyinstall

An Ansible role to install SSH public keys for users on MikroTik RouterOS devices.

---

## 📄 Description

This role manages SSH public keys for local users on MikroTik RouterOS devices (tested on RouterOS 7.19.3). It allows you to upload, validate, and import `.pub` key files to specific MikroTik users via SSH.

It includes optional features such as:

- Verification that users exist on the MikroTik device
- Validation of public key files (format + content)
- Optional flushing of existing SSH keys before reimporting

---

## ✅ Requirements

- MikroTik RouterOS (tested on 7.19.3)
- Ansible 2.9+ with `community.routeros` and `ansible.netcommon` collections
- SSH connectivity configured using:
  - `ansible_connection=network_cli`
  - `ansible_network_os=routeros`

---

## 📝 Role Variables

| Variable                     | Description                                                                                   | Default   |
|-----------------------------|-----------------------------------------------------------------------------------------------|-----------|
| `mikrotik_ssh_users`        | List of users and their public key files                                                      | `[]`      |
| `pubkey_file`               | Path to the `.pub` file (relative to `files/` directory in the role)                          | required per user |
| `pubkeys_path`              | Base directory path where pubkey files are stored                                             | `{{ role_path }}/files` |
| `verify_users_exist`        | Whether to check if each user exists on MikroTik before importing their key                   | `true`    |
| `fail_on_invalid_user`      | If `true`, the role will fail when a user doesn't exist on MikroTik                          | `false`   |
| `verify_pubkeys`            | Whether to validate public key type and syntax before import                                 | `true`    |
| `fail_on_invalid_pubkey`    | If `true`, the role will fail when a public key is missing or invalid                        | `false`   |
| `allow_flushing_pubkeys`    | Whether to remove all SSH keys for users *with keys provided* before importing new ones       | `false`   |

---

## 🔒 Key Flushing Behavior

If `allow_flushing_pubkeys` is set to `true`, existing SSH keys will be removed **only for users defined in `mikrotik_ssh_users` with valid keys**.

> This ensures that users not managed by Ansible retain their keys, avoiding unintentional lockouts or misconfigurations.

MikroTik does **not** allow printing of already imported public key contents. This makes it impossible to determine which keys are currently set for a user. Therefore, **to avoid duplication**, the only reliable way to manage a fixed set of keys is to flush them before reimporting.

If `allow_flushing_pubkeys` is disabled, the same public keys will be duplicated on every run.

---

## 🧪 Key and User Validation

The role can optionally validate that:

- Each target user exists on the MikroTik device
- Each provided `.pub` file exists and is of a valid type:
  - Only `ssh-rsa` and `ssh-ed25519` are supported by MikroTik
- Files that fail validation can be skipped or cause failure based on flags

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

> 📂 Place your public key files (e.g., `John-rsa.pub`, `Tom-ed25519.pub`) inside the `files/` directory of the role.

---

## 🪪 License

This role is distributed under the **MIT-0 License**, which means it is free to use, modify, and distribute without restriction.

---

## 🧑‍💻 Author Information

Paweł Wołochow  
Email: wolochowp@gmail.com

---

## 📌 Notes

- SSH connections to MikroTik devices rely on the `network_cli` connection plugin and the `routeros` network OS setting.
- Key flushing only applies to users provided by Ansible—SSH keys for other users are preserved.
- Validation logic is controlled by `verify_users_exist`, `verify_pubkeys`, and their corresponding failure flags.