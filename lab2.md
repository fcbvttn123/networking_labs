# Table of contents

- [Table of contents](#table-of-contents)
- [Lab Introduction](#lab-introduction)
  - [Multi-Vendor Fact Gathering](#multi-vendor-fact-gathering)
  - [The real-world scenario](#the-real-world-scenario)
- [What I expect to learn](#what-i-expect-to-learn)
- [Folder Structure](#folder-structure)
- [Set up the Ansible environment](#set-up-the-ansible-environment)
  - [Install the required tools](#install-the-required-tools)
  - [Install the vendor collections](#install-the-vendor-collections)
  - [Create the directories](#create-the-directories)
- [Define the multi-vendor inventory](#define-the-multi-vendor-inventory)
  - [Create inventory/hosts.yml](#create-inventoryhostsyml)
  - [Create inventory/group\_vars/cisco\_ios.yml](#create-inventorygroup_varscisco_iosyml)
  - [Create inventory/group\_vars/aruba\_aoscx.yml](#create-inventorygroup_varsaruba_aoscxyml)
  - [Store credentials securely](#store-credentials-securely)


# Lab Introduction

## Multi-Vendor Fact Gathering

This lab simulates a real-world enterprise network where a network automation engineer needs to collect inventory and operational facts from Cisco and Aruba devices, normalize the results, and generate a report without manually logging into every device

## The real-world scenario

- Imagine you are a network automation engineer at a company with several offices

- The company recently acquired another business, and the IT team needs an accurate inventory of the combined network

- The network manager asks you to answer these questions:

    - What devices exist, and which vendor and operating system does each run?

    - What are their hostnames, software versions, models, and serial numbers?

    - Which interfaces exist, and which are operationally up or down?

    - What are the management IP addresses?

    - Can we produce a single inventory report across different vendors?

    - Which devices failed to respond, and which facts could not be collected?


# What I expect to learn

- Organize a multi-vendor Ansible project

- Define devices and connection settings in inventory

- Separate secrets from ordinary configuration

- Gather facts using vendor-specific Ansible network collections

- Handle differences between Cisco IOS XE and Aruba AOS-CX

- Save raw facts and normalize them into a vendor-neutral report

- Validate the results and handle failed connections

- Structure the code so new devices can be added without rewriting the playbook


# Folder Structure

```bash
network-facts-lab/
└── inventory/
│   ├── hosts.yml
│   ├── group_vars/
├── playbooks/
│   ├── gather_facts.yml
├── roles/
│   ├── network_facts/
│   │   ├── tasks/
│   │   ├── defaults/
├── data/
│   ├── mock_facts/
├── reports/
├── ansible.cfg
├── requirements.yml
├── .gitignore
```


# Set up the Ansible environment

## Install the required tools

Install packages

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip git
```

Create the project environment

```bash
mkdir -p ~/labs/network-facts-lab
cd ~/labs/network-facts-lab

python3 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install ansible
```

Verify the installation

```bash
ansible --version
ansible-galaxy --version
```

## Install the vendor collections

Create `requirements.yml`

```yaml
---
collections:
  - name: ansible.netcommon
  - name: cisco.ios
  - name: arubanetworks.aoscx
```

- In Ansible projects, a `requirements.yml` file is used to manage external dependencies—specifically collections and roles — that your playbooks rely on

- Once you have your `requirements.yml` file created in your project root, you can install everything listed inside it by running:

    - For Collections: `ansible-galaxy collection install -r requirements.yml`

    - For Roles: `ansible-galaxy role install -r requirements.yml`

Install the collections

```bash
ansible-galaxy collection install -r requirements.yml
ansible-galaxy collection list # verification
```

## Create the directories

Directories

```bash
mkdir -p inventory/group_vars
mkdir -p playbooks
mkdir -p roles/network_facts/tasks
mkdir -p roles/network_facts/defaults
mkdir -p data/mock_facts
mkdir -p reports
```

Create an `ansible.cfg` file

```ini
[defaults]
inventory = ./inventory/hosts.yml
roles_path = ./roles
host_key_checking = True
retry_files_enabled = False
interpreter_python = auto_silent

[persistent_connection]
connect_timeout = 30
command_timeout = 30
```


# Define the multi-vendor inventory

## Create inventory/hosts.yml

File Creation

```yaml
---
all:
  children:
    network_devices:
      children:
        cisco_ios:
          hosts:
            HQ-RTR01:
              ansible_host: 192.0.2.11
            HQ-SW01:
              ansible_host: 192.0.2.12
            BR-RTR01:
              ansible_host: 192.0.2.14
            BR-SW01:
              ansible_host: 192.0.2.15

        aruba_aoscx:
          hosts:
            HQ-ACC01:
              ansible_host: 192.0.2.13
```

Verification

```bash
# The "ansible-inventory --graph" command is used to display your configured Ansible inventory in a visual ASCII tree structure
$ ansible-inventory --graph

@all:
  |--@network_devices:
  |  |--@aruba_aoscx:
  |  |  |--HQ-ACC01
  |  |--@cisco_ios:
  |  |  |--BR-RTR01
  |  |  |--BR-SW01
  |  |  |--HQ-RTR01
  |  |  |--HQ-SW01
```

## Create inventory/group_vars/cisco_ios.yml

```yaml
---
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: cisco.ios.ios
ansible_user: "{{ vault_network_username }}"
ansible_password: "{{ vault_network_password }}"
```

- Cisco's network_cli connection communicates with network devices over SSH

## Create inventory/group_vars/aruba_aoscx.yml

```yaml
---
# For the legacy REST/httpapi integration.
# Confirm compatibility with your installed AOS-CX collection.
ansible_connection: ansible.netcommon.httpapi
ansible_network_os: arubanetworks.aoscx.aoscx
ansible_user: "{{ vault_network_username }}"
ansible_password: "{{ vault_network_password }}"
ansible_httpapi_use_ssl: true
ansible_httpapi_validate_certs: true
```

## Store credentials securely

Never commit actual usernames and passwords to Git

```bash
ansible-vault create inventory/group_vars/vault.yml
```

Inside the **editor**, add:

```bash
vault_network_username: automation
vault_network_password: "replace-with-the-lab-password"
```