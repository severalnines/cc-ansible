# Ansible Role: ClusterControl (2.x / MCC)

Ansible role to install, configure, and upgrade **Severalnines ClusterControl 2.x (MCC architecture)** on:

- RHEL / Rocky / Alma / CentOS
- Debian / Ubuntu

This role is **idempotent**, **upgrade-safe**, and supports **flexible version pinning**.

---

## Overview

This role performs the following:

- Install ClusterControl Controller (CMON)
- Install MCC UI (`clustercontrol-mcc`)
- Install ClusterControl CLI (`s9s-tools`, always latest)
- Install and configure MySQL/MariaDB on the ClusterControl node
- Initialize CMON database **once**
- Initialize MCC (`ccmgradm`) **once**
- Safe to run multiple times
- Safe to upgrade ClusterControl versions

Default installation mode is **MCC (ClusterControl 2.x)**.

---

## Requirements

- Dedicated host for ClusterControl (recommended)
- Internet access (Severalnines repositories)
- Root access or sudo privileges
- Using Key-based Authentication for SSH / Passwordless SSH

---

## Installation

### Install the role

From GitHub:

```bash
git clone https://github.com/sschyohub/cc-ansible
```

---

## Example Playbook (Recommended)

```yml
---
- hosts: clustercontrol
  become: yes

  vars:
    mysql_root_username: root
    mysql_root_password: admin123

    cmon_mysql_user: cmon
    cmon_mysql_password: admin123

    cc_install_mode: "mcc"
    cc_package_state: "latest"

    clustercontrol_version: "2.3.3"

    mcc_web_port: 443
    mcc_web_root: "/var/www/html/clustercontrol-mcc"

  roles:
    - severalnines.clustercontrol
```

Run:

```bash
ansible-playbook playbook.yml 
or 
ansible-playbook -i inventory/hosts playbook.yml 
```

Access UI:

```
https://<clustercontrol-host>/
```

---

## Version Selection

`clustercontrol_version` supports:

- `latest`
- `2.3.3` (auto-pick per package, fallback to latest)
- `2.3.3-16740` (explicit pin)

CLI packages are always installed as latest.

---

## Idempotency & Upgrades

- MySQL initialization runs once only
- CMON and MCC initialization are marker-guarded
- Safe to re-run playbook
- Safe to upgrade by changing `clustercontrol_version`

---

## Supported Platforms

- RHEL / Rocky / Alma 8–9
- Debian 10–12
- Ubuntu 22.04–24.04

---

## Author

Originally created by Severalnines AB.  
Updated for ClusterControl 2.x MCC architecture.
