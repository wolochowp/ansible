# mikrotik_backup

An Ansible role to create and transfer backups from MikroTik RouterOS devices.

---

## Description

This role automates the backup of MikroTik RouterOS devices (tested on RouterOS 7.19.3) by creating either `.backup` or `.rsc` export files and transferring them to a remote server via SFTP.

It supports two backup transfer modes.

### 🔁 Transfer Modes

#### 1. **Direct Transfer** (`direct_transfer: true`)

- The MikroTik device itself **pushes** the backup file directly to the SFTP server using `/tool fetch upload=yes`.
- Use when:
  - MikroTik has direct access to the remote SFTP server.
  - You want to avoid storing backups temporarily on the Ansible controller.
- ✅ Advantage: More efficient in environments where MikroTik has network access to the destination.
- 🔒 Supports password or key-based SFTP auth.

#### 2. **Indirect Transfer** (`direct_transfer: false`)

- The Ansible controller first **fetches** the backup file from the MikroTik router.
- Then it uploads the file to the SFTP server using `sftp` or `sshpass` (if password-based).
- Use when:
  - MikroTik **cannot reach** the SFTP destination directly (e.g., NAT, firewall).
  - You want central control or post-processing on the Ansible host.
- ✅ Advantage: More control and flexibility, even when MikroTik lacks outbound access.

Password-based or key-based authentication for SFTP is supported, with key-based as the default method.

---

## Requirements

- MikroTik RouterOS (tested on 7.19.3)
- Ansible 2.9+ with `community.routeros` collection installed
- SSH connectivity configured using:
  - `ansible_connection=network_cli`
  - `ansible_network_os=routeros`
- Python packages for SSH on the controller (e.g., `paramiko` or `ansible-pylibssh`)
- `sshpass` installed on the Ansible controller if password-based SFTP authentication is used

---

## Role Variables

Most variables have sane defaults defined in `defaults/main.yml`. Only these are mandatory or typically set in inventory or extra vars:

| Variable                | Description                              | Default            |
|-------------------------|------------------------------------------|--------------------|
| `backup_format`         | Backup file format: `backup` or `rsc`    | `backup`           |
| `direct_transfer`       | Whether MikroTik sends backup directly   | `false`            |
| `remote_backup_host`    | Remote SFTP server hostname/IP            | _must be defined_  |
| `remote_backup_user`    | Remote SFTP username                       | _must be defined_  |
| `remote_backup_path`    | Remote directory to save backup files     | _must be defined_  |
| `remote_backup_port`    | Remote SFTP server port                    | `22`               |
| `remote_backup_password`| Optional password for SFTP (sshpass used) | `undefined`        |
| `non_direct_transfer_tmp_path` | Temporary directory on Ansible controller | `/tmp`        |
| `encrypt_pass`          | Optional encryption password for `.backup` format | `undefined` |

---

## Example Inventory

```ini
[mikrotik]
router1 ansible_host=192.0.2.1

[mikrotik:vars]
ansible_port=22
ansible_user=ansible_user
ansible_connection=network_cli
ansible_network_os=routeros

remote_backup_user=backupuser
remote_backup_host=192.0.2.100
remote_backup_path=/srv/backups/mikrotik
remote_backup_port=22

backup_format=backup
direct_transfer=false
```

## Example Playbook

```yaml
- name: Backup MikroTik router to remote SFTP server
  hosts: mikrotik
  gather_facts: false
  roles:
    - role: mikrotik_backup
      vars:
        remote_backup_password: "{{ vault_sftp_password }}"  # Optional if using SSH key
        encrypt_pass: "{{ vault_backup_encrypt_pass }}"       # Optional encryption for .backup
```
## Handlers

This role provides the following handlers:

- Remove temporary backup file on Ansible controller
Removes the backup file stored temporarily on the Ansible controller after successful upload to the remote server.

- Clean up backup file on MikroTik
Deletes the backup file from the MikroTik device once it has been successfully fetched or uploaded.

---

## License

This role is distributed under the MIT-0 License, which means it is free to use, modify, and distribute without restrictions.

---

## Author Information

Paweł Wołochow
Email: wolochowp@gmail.com

---

## Notes

If using password-based authentication for the remote SFTP server, ensure that sshpass is installed on your Ansible controller.

SSH connections to MikroTik devices rely on the network_cli connection plugin and the routeros network OS setting for proper communication.

Tested with MikroTik RouterOS version 7.19.3.

Store sensitive passwords such as remote_backup_password securely, preferably using Ansible Vault as in example playbook or environment variables.

---