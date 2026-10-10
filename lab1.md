# Table of contents

- [Table of contents](#table-of-contents)
- [Step 1: EVE-NG Topology Setup](#step-1-eve-ng-topology-setup)
- [Step 2: Install Ansible on the Linux VM](#step-2-install-ansible-on-the-linux-vm)
- [Step 3: Create the YAML Project Structure](#step-3-create-the-yaml-project-structure)
  - [Folder Structure](#folder-structure)
  - [Creating files](#creating-files)


# Step 1: EVE-NG Topology Setup

- The **Control Node**: Add an Ubuntu Linux node to the EVE-NG workspace. This will be the Ansible server

- The **Managed Node**: Add a Cisco IOSv or Cisco IOL router to the topology

- Assign static IPs for devices and connect them together

- Make sure the control node can ping the managed nodes


# Step 2: Install Ansible on the Linux VM

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
ansible-galaxy collection install cisco.ios
```


# Step 3: Create the YAML Project Structure

## Folder Structure

```bash
ansible-lab/
├── group_vars/
│   └── cisco_routers.yml
├── inventory.yml
└── playbook.yml
```

## Creating files

Creating a working directory

```bash
mkdir -p ansible-lab/group_vars
cd ansible-lab
```

Create a inventory file `inventory.yml` to define network devices and group them together

```yaml
---
all:
  children:
    cisco_routers:
      hosts:
        router1:
          ansible_host: 192.168.100.50
```

Instead of hardcoding credentials and connection details inside playbooks, create variable files\
Create the file `group_vars/cisco_routers.yml` to store your SSH credentials and tell Ansible how to talk to Cisco IOS devices\
The file name `cisco_routers` must match the group name defined in the `inventory.yml` file

```yaml
---
ansible_user: admin
ansible_password: cisco123
ansible_network_os: cisco.ios
ansible_connection: network_cli
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
```

Create a playbook `playbook.yml` that connects to the router, gathers hardware/software facts, and prints out the device version to the terminal

```yaml
---
- name: Gather and Display Device Information
  hosts: cisco_routers
  gather_facts: false
  tasks:
    - name: Get facts from Cisco IOS device
      cisco.ios.ios_facts:

    - name: Display router hostname and OS version
      ansible.builtin.debug:
        msg: "Success! Router {{ ansible_net_hostname }} is running IOS version {{ ansible_net_version }}"
```

Run the playbook

```bash
$ ansible-playbook -i inventory.yml playbook.yml

PLAY [Gather and Display Device Information] ************************************

TASK [Get facts from Cisco IOS device] ******************************************
ok: [router1]

TASK [Display router hostname and OS version] ***********************************
ok: [router1] => {
    "msg": "Success! Router R1 is running IOS version 15.9(3)M2"
}

PLAY RECAP **********************************************************************
router1                    : ok=2    changed=0    unreachable=0    failed=0
```