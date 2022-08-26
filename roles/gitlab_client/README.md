# Gitlab Client Role

Configures the Ansible Automation Platform to be a gitlab client (used in the Windows Automation scenario `scenario_type: windows`)

Example:

```
- name: Configure GitLab client
  hosts: '*ansible-1'
  become: true
  gather_facts: true
  tags:
    - git
  tasks:
    - include_role:
        name: ansible.scenarios.gitlab_client
```
