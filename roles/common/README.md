# Common Role

This is the scenario common role (for Linux nodes, specifically RHEL)


Sets up the following:

    - ssh
    - sudo
    - common

Example

```
- name: Configure common options on managed nodes and control nodes
  hosts: "managed_nodes:control_nodes"
  gather_facts: false
  become: true
  tasks:
    - include_role:
        name: ansible.scenarios.user_accounts
    - include_role:
        name: ansible.scenarios.common
```
