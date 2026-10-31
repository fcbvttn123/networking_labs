# Folder Structure

- ansible.cfg `# must be the same name ansible.config`

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
```


# Set credentials for `invnetory/group_vars/ios.yml`

`yaml` file

```yaml
---
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: cisco.ios.ios

ansible_user: "{{ lookup('env', 'NETWORK_USERNAME') }}"
ansible_password: "{{ lookup('env', 'NETWORK_PASSWORD') }}"

# Enable this only if your devices require enable mode.
ansible_become: true
ansible_become_method: enable
ansible_become_password: "{{ lookup('env', 'NETWORK_ENABLE_PASSWORD') }}"

backup_directory: "{{ playbook_dir }}/output/backups"
report_directory: "{{ playbook_dir }}/output"
```

Set the credentials in your shell rather than committing passwords to Git

```bash
export NETWORK_USERNAME='your_lab_username'
read -s -p "SSH password: " NETWORK_PASSWORD
export NETWORK_PASSWORD
echo
```