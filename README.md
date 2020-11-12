ReaR
=========

This role is used to create backups using ReaR ( Relax and Recover - https://relax-and-recover.org/ ).


Requirements
------------

This is not strict requirements and it may not work with other versions than tested ones.
Anyway. feel free to test by yourself, suggest addition of new functionality and contribute.

Role is tested with:
- Ansible version >= 2.8.6
- CentOS version >= 7.6 (1803)
- Backup methods:
  - rsync


Role Variables
--------------

Variables and their descriptions copied from defaults/main.yml

```bash

# If you're authenticated to managed hosts via password, define these variables.
# If your ssh user is not root, it must be in sudoers group ("wheel")
# and "ansible_become_pass" variable must be the same as "ansible_ssh_pass",
# otherwise root password must be provided via "ansible_become_pass" variable.
# It's strongly recommended to disable password authentication and use ssh keypair,
# If this is not possible, it's recommended to encrypt your password(s) via Ansible Vault like this:
# ansible-vault encrypt-string -n ansible_ssh_pass "SECURE_SSH_PASSWORD"
# ansible-vault encrypt_string -n ansible_become_pass "SECURE_SSH_PASSWORD"
# Then replace both variables inside needed files in group_vars/ and host_vars/
# directories with output:
ansible_ssh_pass: ""
ansible_become_pass: ""


# Project name needed to logically separate group of hosts,
# used to create folders that contain directories
# with different environments:
project_name: test

# Environment name needed to logically separate group of hosts that belongs
# to single environment of the project ("development", "staging", "production" etc),
# used as directory name to create and store ssh keypairs in:
project_environment: development


# Arguments passed to 'rear mkbackup command' before starting
# backup process:
rear_mkbackup_args: -d


# Host used to store backups, string can be DNS name of IP address:
rear_rsync_remote_host: "{{ hostvars[groups.backuphosts[0]].ansible_default_ipv4.address }}"

# This user must be allowed to login via ssh on backup host,
# and have write access to rear_rsync_remote_path directory, pay attention that
# this is tested only with root user, so using the other one can cause problems:
rear_rsync_remote_user: root

# Absolute path of directory on backup host, which will store files needed for restore:
rear_rsync_remote_path: /var/lib/rear

# Absolute paths of files which must be excluded from backup, separated by whitespace:
rear_files_to_exclude: "'/var/log/lastlog'"
# Due to backup problems which are caused by incorrectly displayed
# size of "var/log/lastlog" file, it will be excluded from backup by default,
# if you want to backup this file, you'll need to do it manually
# or write task/playbook which manually backups this file.


# Array of systemd service names, which will be stopped and disabled before backup, to prevent
# problems with unfinished work of applications, databases and other services. These services
# will be stopped on every host. If you want to stop different services on different ansible groups
# or hosts, you must create directories/files for group and host variables, and define it there:
rear_services_to_stop: []

# If stopped services must be also disabled at next reboot or not. This variable is usually
# used when migration is performed, and there is no need to start services automatically,
# if you want first check if OS is properly migrated or not:
rear_disable_services: false


# If value of this variable is true, then mapping directory and files will be created
# to change ip address(es), subnet mask(s) and route(s) which will be present on rescue system.
# This mostly needed when rescue environment started in network that differs from original one:
rear_mappings: false

# Network device name for use in recover environment:
rear_mappings_dev: "{{ ansible_default_ipv4.interface }}"

# Network interface ip for use in recover environment:
rear_mappings_ip: "{{ ansible_default_ipv4.address }}"

# Network interface subnet mask for use in recover environment:
rear_mappings_subnet_mask: "{{ ansible_default_ipv4.netmask }}"

# Default gateway's ip for use in recover environment:
rear_mappings_gateway: ""

```


Dependencies
------------

none


Example Playbook
----------------

```bash
---
- hosts: localhost
  gather_facts: false
  become: no
  tasks:
  - name: Check ansible version >=2.8.6
    assert:
      msg: Ansible must be v2.8.6 or higher
      that:
      - ansible_version.string is version("2.8.6", ">=")
    tags:
    - check
  vars:
    ansible_connection: local

- hosts: all
  become: yes
  tasks:
  - import_role:
      name: rear

```

More detailed examples ( inventories, playbooks etc. ) of this and other roles can be found [here](https://github.com/caermeglaeddyv/examples/tree/dev/ansible).

It's highly recommended to start you test deploys from there, especially if you use Google Cloud Platform or VMware vCenter as your infrastructure, for now that [repository](https://github.com/caermeglaeddyv/examples) contains [Packer](https://github.com/caermeglaeddyv/examples/tree/dev/packer) and [Terraform](https://github.com/caermeglaeddyv/examples/tree/dev/terraform) examples to build templates and deploy machines on this platforms.


License
-------

[Apache 2.0](https://github.com/caermeglaeddyv/ansible-role-rear/blob/dev/LICENSE)


Author Information
------------------

Copyright 2020 [caermeglaeddyv](https://github.com/caermeglaeddyv)
