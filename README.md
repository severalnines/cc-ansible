# Ansible Role: ClusterControl

Installs and configures Severalnines ClusterControl Latest version on RHEL/CentOS or Debian/Ubuntu servers. This role is **idempotent**, **upgrade-safe**, and also supports **flexible version pinning** e.g version `2.3.3`. The following package that will installed by this role is :
  - clustercontrol-mcc
  - clustercontrol-notifications
  - clustercontrol-ssh
  - clustercontrol-cloud
  - clustercontrol-clud
  - clustercontrol-controller
  - clustercontrol-proxy
  - clustercontrol-kuber-proxy
  - s9s-tools


## Overview

Installs ClusterControl for your new database node/cluster deployment or on top of your existing database node/cluster. ClusterControl is a management and automation software for database clusters. It helps deploy, monitor, manage and scale your database cluster.

Supported database clusters:

 - MySQL/MariaDB Replication
 - Percona XtraDB Cluster
 - MariaDB Cluster (Galera)
 - MySQL Cluster
 - MongoDB Replica Set
 - MongoDB Sharded Cluster
 - PostgreSQL Streaming Replication
 - TimescaleDB Streaming Replication

More details at [Severalnines](http://www.severalnines.com) website.


---

## Requirements

- ClusterControl server must run on a clean dedicated host with internet connection.
- Root access or sudo privileges
- Using Key-based Authentication for SSH / Passwordless SSH

---

## Installation

### Setup the role
1) To create an Ansible directory with a minimal configuration in a custom location, use the following example:
   
   ```bash
   mkdir -p ~/ansible-clustercontrol
   mkdir -p ~/ansible-clustercontrol/inventory
   mkdir -p ~/ansible-clustercontrol/roles
   touch ~/ansible-clustercontrol/ansible.cfg
   touch ~/ansible-clustercontrol/inventory/hosts
   touch ~/ansible-clustercontrol/playbook.yml
   cd ~/ansible-clustercontrol
   ``` 
    Alternatively, you can use the default Ansible directory: `/etc/ansible/`.

2) Get the ClusterControl Ansible role from Ansible Galaxy or Github.  

      - Ansible Galaxy (always stable from master branch):

          ```bash
          ansible-galaxy install severalnines.cc-ansible
          ```

      - Github (*master branch*):

          ```bash
          git clone https://github.com/severalnines/cc-ansible
          ```
      - Copy the to the role directory (`/etc/ansible/`)
          ```bash
          cp -rf cc-ansible /etc/ansible/roles/cc-ansible 
          ```
          or if you use custom directory
          ```bash
          cp -rf cc-ansible ~/ansible-clustercontrol/roles/cc-ansible 
          ```
   

3) Create playbooks. Refer to the [Example Playbook](#example-playbook) section or [examples](/tree/master/examples) directory.

4) Run the playbook.

    ```bash
    ansible-playbook playbook.yml 
    or 
    ansible-playbook -i inventory/hosts playbook.yml
    ```

---
### Example Playbook    

- Example Playbook for installation.  
  
  ```
  - hosts: clustercontrol
    become: yes

    vars:
      mysql_root_username: root
      mysql_root_password: admin123

      cmon_mysql_user: cmon
      cmon_mysql_password: admin123

      cc_install_mode: "mcc"
      cc_package_state: "latest"

      clustercontrol_version: "latest"  # To install a specific version, replace "latest" with the version number (e.g., "2.3.3"). Refer to the release notes for available versions.

      mcc_web_port: 443
      mcc_web_root: "/var/www/html/clustercontrol-mcc"

    roles:
      - severalnines.cc-ansible
  ```
Access UI:  
- Once the installation is complete, you can access the ClusterControl UI at : 

  ```
  https://<clustercontrol-host>/
  ```

---

## Usage  

This role is only capable of installing ClusterControl version `2.2` and above, which utilizes MCC as the web server. Once the installation is complete you require to create the default admin user by specifying a username (username "admin" is reserved) and password on the welcome page.

### Using Vault in playbooks
The following example shows the steps to use Ansible Vault to encrypt the ``mysql_root_password`` and ``cmon_mysql_password`` configured on the ClusterControl host.

- ``mysql_root_password``: admin123
- ``cmon_mysql_password``: admin123

1) Create a password file to encrypt variables on the host where the playbook resides, called ``password-file``:

    ```bash
    $  echo -n 'admin123' > password-file
    ```
      

2) Generate the Vault value for ``mysql_root_password`` variable using the same password file:

    ```bash
    $ echo -n 'admin123' | ansible-vault encrypt_string --vault-id dev@password-file --stdin-name 'mysql_root_password'
    Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a newline)

    Encryption successful
    mysql_root_password: !vault |
              $ANSIBLE_VAULT;1.2;AES256;dev
              63656662666337663062373532313264366135643732323166666563666563653036363666346432
              6335643637313630646565633337326663383038663830640a373433633265366337323839323134
              63613139633965653331626362333734333134393463666336616332636562376532316230336434
              6638323630663066380a343332643537623864653161363839306131333438626534336634313364
              3339
    ```

3) Generate the Vault value for ``cmon_mysql_password`` variable using the same password file:

    ```bash
    $ echo -n 'admin123' | ansible-vault encrypt_string --vault-id dev@password-file --stdin-name 'cmon_mysql_password'
    Reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a newline)

    Encryption successful
    cmon_mysql_password: !vault |
              $ANSIBLE_VAULT;1.2;AES256;dev
              62623633643436316135656466303162356239396664396563316366363937366630633433653833
              3061623439633563663363393163366562633531376532350a616132386261626565613264303030
              63343936333266386234316665643334366136346462386537626165613835353935316139663863
              3936326639363137650a643631623364333035303365636538386639643235643439363566636530
              6435
    ```

4) Then add the vaults variables into the playbook:

    ```yml
    - hosts: cc
      become: yes

      vars:
        #credentials
        mysql_root_password: !vault |
              $ANSIBLE_VAULT;1.2;AES256;dev
              63656662666337663062373532313264366135643732323166666563666563653036363666346432
              6335643637313630646565633337326663383038663830640a373433633265366337323839323134
              63613139633965653331626362333734333134393463666336616332636562376532316230336434
              6638323630663066380a343332643537623864653161363839306131333438626534336634313364
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

        clustercontrol_version: "latest" #"2.3.3"

        mcc_web_port: 443
        mcc_web_root: "/var/www/html/clustercontrol-mcc"

      roles:
        - cc-ansible
    ```

5) Run the playbook with correct vault ID:

    ```bash
    $ ansible-playbook --vault-id=dev@password-file playbook.yml
    ```
    Or
    ```bash
     ansible-playbook --vault-id=dev@password-file -i inventory/hosts playbook.yml
    ```

---
### Version Selection

To select a specific version, you can define the `clustercontrol_version` variable in your playbook. Supported values include:

- `latest` : Installs the most recent version available.
- Specific version (e.g., `2.3.3`): Picks the designated package, with a fallback to latest if not found. Please refer to the [release notes]( https://docs.severalnines.com/clustercontrol/latest/release-notes/) page.

Please note that the s9s CLI packages will always be installed as the latest version.

---

## Idempotency & Upgrades
This role supports idempotency and ClusterControl upgrades, simply adjust the `clustercontrol_version` value in your playbook to proceed. To ensure a safe and consistent execution, the process follows these rules:

- MySQL initialization runs once only during first installation.
- CMON and MCC initialization are marker-guarded
- Safe to re-run playbook
- Safe to upgrade by changing `clustercontrol_version`

---

## Supported Platforms
This role is built on top of Ansible v2.17.14 and compatible on the following operating systems :

- Red Hat Enterprise Linux 8.x/9.x
- Rocky Linux 8.x/9.x
- AlmaLinux 8.x/9.x 
- Ubuntu 22.04/24.04 LTS
- Debian 10.x/11.x/12.x

---

## Author

Originally created by Severalnines AB.  
Updated for ClusterControl 2.x MCC architecture.
