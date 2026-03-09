# Ansible Role: ClusterControl

Installs and configures Severalnines ClusterControl on RHEL/CentOS or Debian/Ubuntu servers.
This role is **idempotent**, **upgrade-safe**, and supports **flexible version pinning** (e.g. `2.3.3`).

The following packages will be installed by this role:

- `clustercontrol-controller`
- `clustercontrol-mcc`
- `clustercontrol-proxy`
- `clustercontrol-ssh`
- `clustercontrol-cloud`
- `clustercontrol-notifications`
- `clustercontrol-clud`
- `clustercontrol-kuber-proxy`
- `s9s-tools`


## Overview

Installs ClusterControl for new database deployments or on top of existing database clusters.
ClusterControl is a management and automation platform for database clusters — it helps deploy, monitor, manage, and scale your database infrastructure.

Supported database clusters:

- MySQL/MariaDB Replication
- Percona XtraDB Cluster
- MariaDB Cluster (Galera)
- MySQL Cluster
- MongoDB Replica Set
- MongoDB Sharded Cluster
- PostgreSQL Streaming Replication
- TimescaleDB Streaming Replication

More details at the [Severalnines website](https://www.severalnines.com).


## Requirements

- ClusterControl server must run on a **clean dedicated host** with internet access.
- Root access or sudo privileges.
- Key-based (passwordless) SSH authentication.


## Installation

1) Create an Ansible working directory:

```bash
mkdir -p ~/ansible-clustercontrol/{inventory,roles}
touch ~/ansible-clustercontrol/ansible.cfg
touch ~/ansible-clustercontrol/inventory/hosts
touch ~/ansible-clustercontrol/playbook.yml
cd ~/ansible-clustercontrol
```

Alternatively, use the default Ansible directory: `/etc/ansible/`.

2) Get the ClusterControl Ansible role from Ansible Galaxy or Github.

Ansible Galaxy (always stable from master branch):

```bash
ansible-galaxy install severalnines.cc-ansible
```

Github (master branch):

```bash
git clone https://github.com/severalnines/cc-ansible
```

Copy to the roles directory:

```bash
# Default Ansible directory
cp -rf cc-ansible /etc/ansible/roles/cc-ansible

# Or custom directory
cp -rf cc-ansible ~/ansible-clustercontrol/roles/cc-ansible
```

3) Create the Ansible inventory file and save it in the `inventory/` folder.

```ini
[clustercontrol]
your-cc-host ansible_user=root
```

4) Create a playbook. Refer to the Example Playbook section below.

5) Run the playbook.

```bash
ansible-playbook -i inventory/hosts playbook.yml
```


## Example Playbook

```yaml
- hosts: clustercontrol
  become: yes

  vars:
    mysql_root_username: root
    mysql_root_password: admin123

    cmon_mysql_user: cmon
    cmon_mysql_password: admin123

    cc_install_mode: "mcc"
    cc_package_state: "latest"

    # Use "latest" or pin to a specific version e.g. "2.3.3"
    clustercontrol_version: "latest"

    mcc_web_port: 443
    mcc_web_root: "/var/www/html/clustercontrol-mcc"

  roles:
    - severalnines.cc-ansible
```

Once installation is complete, access the ClusterControl UI at:

```
https://<clustercontrol-host>/
```

**Note:** On first login, create an admin account on the welcome page. The username `admin` is reserved — use a different username.


## Usage

This role installs ClusterControl `2.2` and above, which uses MCC as the web server.

### Version Selection

Set the `clustercontrol_version` variable in your playbook. Supported values:

- `"latest"` — Installs the most recent available version.
- Specific version (e.g. `"2.3.3"`) — Installs the designated package, with a fallback to latest if not found.

**Note:** The `s9s-tools` package is always installed as the latest version regardless of `clustercontrol_version`.

See the [release notes](https://docs.severalnines.com/clustercontrol/latest/release-notes/) for available versions.

### Using Vault in playbooks

It is strongly recommended to encrypt sensitive variables using Ansible Vault instead of storing plaintext passwords in your playbook.

Variables you may want to encrypt:

- `mysql_root_password`
- `cmon_mysql_password`

1) Create a password file on the host where the playbook resides:

```bash
echo -n 'myVaultPassword' > password-file
```

2) Generate the Vault value for `mysql_root_password`:

```bash
echo -n 'admin123' | ansible-vault encrypt_string \
  --vault-id dev@password-file \
  --stdin-name 'mysql_root_password'
```

3) Generate the Vault value for `cmon_mysql_password`:

```bash
echo -n 'admin123' | ansible-vault encrypt_string \
  --vault-id dev@password-file \
  --stdin-name 'cmon_mysql_password'
```

4) Add the encrypted values to your playbook:

```yaml
- hosts: clustercontrol
  become: yes

  vars:
    mysql_root_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          63656662666337663062373532313264366135643732323166666563666563653036363666346432
          6335643637313630646565633337326663383038663830640a373433633265366337323839323134
          63613139633965653331626362333734333134393463666336616332636562376532316230336434
          6638323630663066380a343332643537623864653161363839306131333448626534336634313364
          3339

    cmon_mysql_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          62623633643436316135656466303162356239396664396563316366363937366630633433653833
          3061623439633563663363393163366562633531376532350a616132386261626565613264303030
          63343936333266386234316665643334366136346462386537626165613835353935316139663863
          3936326639363137650a643631623364333035303365636538386639643235643439363566636530
          6435

    cc_install_mode: "mcc"
    cc_package_state: "latest"
    clustercontrol_version: "latest"
    mcc_web_port: 443
    mcc_web_root: "/var/www/html/clustercontrol-mcc"

  roles:
    - cc-ansible
```

5) Run the playbook with the vault ID:

```bash
ansible-playbook --vault-id=dev@password-file -i inventory/hosts playbook.yml
```


## Idempotency & Upgrades

This role supports idempotency and ClusterControl upgrades. To ensure a safe and consistent execution, the process follows these rules:

- MySQL initialization runs only once during first installation.
- CMON and MCC initialization are marker-guarded.
- Safe to re-run the playbook.
- Safe to upgrade by changing `clustercontrol_version`.


## Supported Platforms

This role is built on top of Ansible `v2.17.14` and is compatible on the following operating systems:

- Red Hat Enterprise Linux 8.x / 9.x
- Rocky Linux 8.x / 9.x
- AlmaLinux 8.x / 9.x
- Ubuntu 22.04 / 24.04 LTS
- Debian 10.x / 11.x / 12.x


## Author

Originally created by [Severalnines AB](https://www.severalnines.com).
Updated for ClusterControl 2.x MCC architecture.
