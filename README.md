# Pi Media Stack Automation

Ansible configuration for preparing Raspberry Pi 5 devices running Debian Bookworm
for a media stack. The playbooks now cover system preparation, mounting of
storage/download drives, creation of the required folder structure, Docker
deployment, mount guards, optional Tailscale connectivity, Samba shares, and
app-specific extras.

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
  docker_engine/
    tasks/main.yml
  compose_deploy/
    tasks/main.yml
  mount_guard/
    tasks/main.yml
    handlers/main.yml
  tailscale/
    tasks/main.yml
  samba_shares/
    defaults/main.yml
    tasks/main.yml
    handlers/main.yml
  app_extras/
    tasks/main.yml
    templates/get_iplayer.config.json.j2
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
