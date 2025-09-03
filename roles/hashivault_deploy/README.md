# HashiVault Deploy - Ansible Role

## Overview
`hashivault_deploy` is an Ansible role to install, configure, and bootstrap HashiCorp Vault clusters on Linux hosts. It supports multiple storage backends, seal types (including auto-unseal), TLS, and mTLS configurations. This role is modular, idempotent, and designed for production-ready deployments.

---

## Features
- Installs Vault from official releases
- Supports Raft (recommended) and File storage backends
- Supports Shamir and cloud KMS-based auto-unseal
- TLS and mTLS support with wildcard or per-node certificates
- Systemd hardening options for secure deployment
- Dynamic cluster join via `retry_join`
- Vault token and recovery key management
- Fully customizable via role variables

---

## Requirements
- Ansible 2.15+ (or latest stable)
- Linux hosts (Debian, RHEL/CentOS)
- SSH access with `become: true`

Optional for auto-unseal:
- AWS KMS / Azure Key Vault / GCP KMS credentials
- HSM / Transit engine if used

---

## Role Variables
All role variables have sane defaults and can be overridden per host or group.

### System & Installation Defaults
```yaml
hashivault_deploy_systemd_service_name: vault.service
hashivault_deploy_user: vault
hashivault_deploy_group: vault
hashivault_deploy_version: latest
hashivault_deploy_config_dir: /etc/vault.d
hashivault_deploy_data_dir: /opt/vault/data
hashivault_deploy_cluster_name: vault-test-cluster
hashivault_deploy_telemetry: {}
```

### TLS / HTTPS Defaults
```yaml
hashivault_deploy_tls_enabled: true
hashivault_deploy_mtls_enabled: true
hashivault_deploy_tls_cert_dir: /etc/vault/tls
hashivault_deploy_tls_src_dir: '{{ role_path }}/files'
hashivault_deploy_tls_wildcard: false
hashivault_deploy_tls_ca_file: ca.pem
```

### Vault Core Defaults
```yaml
hashivault_deploy_ui_enabled: true
hashivault_deploy_disable_mlock: false
hashivault_deploy_storage_type: raft
hashivault_deploy_storage_config:
  path: '{{ hashivault_deploy_data_dir }}'
  node_id: '{{ inventory_hostname }}'
  retry_join: []
```

### Seal Configuration
Supported seal types: `shamir`, `awskms`, `azurekeyvault`, `gcpckms`, `hsm`, `transit`
```yaml
hashivault_deploy_seal_type: shamir
hashivault_deploy_seal_config: {}
```

#### Example Snippets for Auto-Unseal
```yaml
# AWS KMS
# hashivault_deploy_seal_type: awskms
# hashivault_deploy_seal_config:
#   region: eu-north-1
#   kms_key_id: arn:aws:kms:REGION:ACCOUNT_ID:key/EXAMPLE-KEY-ID
#   access_key: <AWS_ACCESS_KEY_ID>
#   secret_key: <AWS_SECRET_ACCESS_KEY>
```

(See full role defaults for Azure, GCP, HSM, Transit examples.)

### Vault Token & Recovery Key Management
```yaml
hashivault_deploy_token_save_path: /secrets
hashivault_deploy_token_save_filename: '{{ hashivault_deploy_cluster_name }}-vault-init-keys.json'
hashivault_deploy_shamir_key_shares: 5
hashivault_deploy_shamir_key_threshold: 3
hashivault_deploy_stored_shares: 1
hashivault_deploy_enable_recovery: true
hashivault_deploy_recovery_key_shares: 5
hashivault_deploy_recovery_key_threshold: 3
```

### Dynamic Defaults
```yaml
hashivault_deploy_vault_ip: '{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}'
hashivault_deploy_dns_name: '{{ ansible_fqdn }}'
hashivault_deploy_bind_address: '{{ hashivault_deploy_vault_ip }}:{{ hashivault_deploy_vault_port }}'
hashivault_deploy_protocol: '{{ "https" if hashivault_deploy_tls_enabled else "http" }}'
hashivault_deploy_api_addr: '{{ hashivault_deploy_protocol }}://{{ hashivault_deploy_dns_name }}:{{ hashivault_deploy_vault_port }}'
hashivault_deploy_cluster_addr: '{{ hashivault_deploy_protocol }}://{{ hashivault_deploy_dns_name }}:{{ hashivault_deploy_vault_cluster_port }}'
```

---

## Example Inventory
```ini
[debians]
vault1 ansible_host=192.168.1.50 hashivault_deploy_vault_ip=192.168.1.50 hashivault_deploy_dns_name=vault1.example.local
vault2 ansible_host=192.168.1.51 hashivault_deploy_vault_ip=192.168.1.51 hashivault_deploy_dns_name=vault2.example.local
vault3 ansible_host=192.168.1.52 hashivault_deploy_vault_ip=192.168.1.52 hashivault_deploy_dns_name=vault3.example.local

[debians:vars]
ansible_user=ansible
```

---

## Example Playbook
```yaml
- name: Install Vault
  hosts: debians
  gather_facts: true
  become: true
  roles:
    - role: hashivault_deploy
```

---

## License
MIT-0
