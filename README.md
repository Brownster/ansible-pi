# Pi Media Stack Automation

Ansible configuration for preparing Raspberry Pi 5 devices running Debian Bookworm
for a media stack. The playbooks currently cover system preparation, mounting of
storage/download drives, and creation of the required folder structure.

## Repository layout

```
inventory/
  hosts.yml
  group_vars/
    all.yml
    vault.yml      # encrypt with `ansible-vault`
roles/
  system_prep/
    tasks/main.yml
  mounts/
    tasks/main.yml
  folders/
    tasks/main.yml
playbooks/
  site.yml
files/
  .gitkeep
```

## Usage

Run the main playbook against your inventory:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/site.yml --ask-become-pass
```

Add `--ask-vault-pass` if `inventory/group_vars/vault.yml` has been encrypted.

The roles are heavily commented so they can be extended later with Docker and
application deployment tasks.
