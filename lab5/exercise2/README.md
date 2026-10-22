# Folder Structure

- ansible.cfg

- inventory/
    - hosts.yml
    - group_vars/
        - ios.yml

- site.yml (main playbook)

- requirements.yml (collection dependencies)

- roles/
    - gather_facts/
        - tasks/
            - main.yml
    - backup_config/
        - tasks/
            - main.yml
    - report/
        - tasks/
            - main.yml
        - templates/
            - inventory.html.j2

- output/


# Install Ansible dependencies

Create requirements.yml:

```yaml
---
collections:
  - name: cisco.ios
  - name: ansible.netcommon
```

Install them:

```bash
ansible-galaxy collection install -r requirements.yml

# check the installation
ansible-galaxy collection list
ansible --version