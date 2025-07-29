# wolochowp.mikrotik.backup

An Ansible role to create and transfer backups from MikroTik RouterOS devices.

---

## Description

This role automates the backup of MikroTik RouterOS devices (tested on RouterOS 7.19.3) by creating either `.backup` or `.rsc` export files and transferring them to a remote server via SFTP. It supports both:

- **Direct transfer**: MikroTik uploads the backup file directly to the remote SFTP server.
- **Indirect transfer**: Backup is fetched to the Ansible controller first, then uploaded via SFTP.

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
mikrotik ansible_host=192.168.1.1

[mikrotik:vars]
ansible_port=22
ansible_user=ansible_automation
ansible_connection=network_cli
ansible_network_os=routeros


