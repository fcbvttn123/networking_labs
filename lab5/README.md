# Project Structure

```bash
lab5/
├── inventory.yml
├── site.yml
├── group_vars/
│   └── datacenter_switches.yml
└── roles/
    └── network_baseline/
        ├── defaults/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── templates/
            └── global_config.j2
            └── vlans.j2
```


# Install requried collections for `role` and `j2`

```bash
ansible-galaxy collection install cisco.ios ansible.netcommon
```


# Validate and Run

```bash
# Check inventory
ansible-inventory -i inventory.yml --graph

# Check playbook syntax
ansible-playbook -i inventory.yml site.yml --syntax-check

# Test connectivity
ansible datacenter_switches -i inventory.yml \
  -m cisco.ios.ios_command \
  -a '{"commands":["show version"]}'

# Run the role
ansible-playbook -i inventory.yml site.yml --ask-vault-pass