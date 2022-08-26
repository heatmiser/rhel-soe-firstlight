# Control Node Role

This role will setup control nodes for Ansible Automation scenarios (same for all scenario types)

Example:

```
- name: configure ansible control node
  hosts: '*ansible-1'
  gather_facts: true
  become: true
  tasks:
    - include_role:
        name: ansible.scenarios.control_node
```
