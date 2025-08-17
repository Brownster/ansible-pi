# Pi Media Stack Automation

Ansible configuration for preparing Raspberry Pi 5 devices running Debian Bookworm
for a media stack. The playbooks now cover system preparation, mounting of
storage/download/backup drives, creation of the required folder structure, Docker
deployment, controlled container updates, automated backups, monitoring exporters,
optional Tailscale connectivity, Samba shares, and
app-specific extras. Docker configuration lives under `/opt/mediastack`
(overridable via the `mediastack_root` variable) to keep the stack
self-contained.

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
  backup_mount/
    tasks/main.yml
  folders/
    tasks/main.yml
  docker_engine/
    tasks/main.yml
  watchtower_policy/
    tasks/main.yml
    templates/watchtower.env.j2
  compose_deploy/
    tasks/main.yml
  backups/
    tasks/main.yml
    templates/backup_docker.sh.j2
  monitoring/
    tasks/main.yml
  webui_integration/
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

## Backups and monitoring

The backup role installs `/usr/local/bin/backup_docker.sh` and a systemd timer
(`docker-backup.timer`) that archives Docker configuration and manifests under
`/mnt/backups/<hostname>/YYYYMMDD`. The latest snapshot is mirrored to
`/opt/mediastack/backups` for consumption by the web UI.

Key variables in `inventory/group_vars/all.yml`:

- `backups_device_uuid` – UUID of the USB drive mounted at `/mnt/backups`
- `backup_keep_daily` / `backup_keep_weekly` – retention policy
- `backup_timer_on_calendar` – systemd time specification
- `watchtower_schedule` – when Watchtower checks for image updates
- `mediastack_root` – base directory holding Docker configs and compose files

To restore a backup, copy the desired dated folder back to
`/opt/mediastack` and run `docker compose up -d`.

The monitoring role exposes host metrics via `node_exporter` on port 9100 and
container metrics via `cAdvisor` on port 8082.
