# Pi Media Stack Automation

This Ansible project automates the setup and deployment of a comprehensive media stack on Raspberry Pi 5 devices running Debian Bookworm. It's designed to be a repeatable and idempotent way to get a full-featured media server up and running with minimal manual intervention.

## Features

*   **System Preparation:** Prepares the base Debian system with necessary packages, a dedicated user, and correct timezone settings.
*   **Docker Engine Installation:** Installs Docker and the Docker Compose plugin.
*   **Automated Storage Mounting:** Mounts storage, download, and backup drives using their UUIDs.
*   **Dockerized Media Stack:** Deploys a full suite of media applications using Docker Compose, including:
    *   **Arr Stack:** Sonarr, Radarr, Lidarr, and Jackett for managing your media.
    *   **Download Clients:** Transmission and SABnzbd.
    *   **Media Server:** Jellyfin.
    *   **And more:** See the `docker-compose.yml.j2` file for a full list of services.
*   **VPN Integration:** Includes Gluetun for routing traffic through a VPN.
*   **Automated Backups:** Sets up a systemd timer to automatically back up your Docker configurations.
*   **Monitoring:** Deploys Node Exporter and cAdvisor for monitoring host and container metrics.
*   **Samba Shares:** Configures Samba for easy network access to your media.
*   **Tailscale Integration:** Optionally installs Tailscale for secure remote access.

## Requirements

*   **Hardware:** A Raspberry Pi 5 is recommended.
*   **Operating System:** Debian Bookworm (or a compatible Debian-based OS).
*   **Ansible:** Ansible 2.10 or later installed on your control machine.
*   **Storage:** At least one external drive for your media and another for downloads is recommended.

## Configuration

Configuration is managed through variables in the `inventory/group_vars` directory.

### `all.yml`

This file contains non-sensitive configuration options. Here are some of the most important variables:

*   `main_user` and `main_group`: The username and group for the dedicated media user.
*   `puid` and `pgid`: The user and group ID for the media user.
*   `storage_device_uuid`, `downloads_device_uuid`, `backups_device_uuid`: The UUIDs of your storage, download, and backup drives.
*   `mediastack_root`: The root directory for all Docker configurations.
*   `set_hostname`: Whether to set the hostname of the device.
*   `hostname_value`: The hostname to set.

### `vault.yml`

This file should be encrypted with `ansible-vault` and used to store sensitive information like API keys and passwords. For example, your VPN credentials should be stored here.

To encrypt the file, run:

```bash
ansible-vault encrypt inventory/group_vars/vault.yml
```

### Docker Images and Ports

The Docker images and ports for all services are defined as variables in `inventory/group_vars/all.yml`. This allows you to easily change the image tags or port mappings without modifying the `docker-compose.yml.j2` template.

## Usage

1.  **Clone the repository:**

    ```bash
    git clone <repository-url>
    cd ansible-pi
    ```

2.  **Configure your inventory:**

    *   For physical Raspberry Pis, edit `inventory/hosts.yml` to add the device's IP address or hostname.
    *   For local Vagrant testing, `inventory/vagrant.yml` already defines a `default` host for the VM.
    *   Edit `inventory/group_vars/all.yml` to customize the configuration.
    *   Create and encrypt `inventory/group_vars/vault.yml` to store your secrets.

3.  **Run the playbook:**

    To deploy to a physical Raspberry Pi:

    ```bash
    ansible-playbook -i inventory/hosts.yml playbooks/site.yml --ask-become-pass
    ```

    If you've encrypted your `vault.yml` file, you'll also need to add the `--ask-vault-pass` flag:

    ```bash
    ansible-playbook -i inventory/hosts.yml playbooks/site.yml --ask-become-pass --ask-vault-pass
    ```

    The Vagrant setup uses `inventory/vagrant.yml` automatically when you run `vagrant up` or `vagrant provision`.

## Local Testing with Vagrant

Instead of testing on a physical Raspberry Pi, you can use Vagrant to create a local virtual machine that mimics your Pi's environment. This is a much faster and more convenient way to test your Ansible playbook.

The provided `Vagrantfile` points to `inventory/vagrant.yml`, which contains a `default` host definition for the virtual machine.

### Requirements

*   [Vagrant](https://www.vagrantup.com/downloads)
*   [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

### Usage

1.  **Start the test environment:**

    ```bash
    vagrant up
    ```

    This command will download a Debian image, create a virtual machine, and run the Ansible playbook against it.

2.  **Re-run the playbook:**

    ```bash
    vagrant provision
    ```

    If you make changes to your playbook, you can re-run it against the existing virtual machine with this command.

3.  **Destroy the test environment:**

    ```bash
    vagrant destroy
    ```

    When you're done testing, you can destroy the virtual machine to free up resources.

## Roles

This project is organized into the following Ansible roles:

*   **`system_prep`:** Prepares the base system.
*   **`docker_engine`:** Installs Docker and Docker Compose.
*   **`mounts`:** Mounts the storage and download drives.
*   **`backup_mount`:** Mounts the backup drive.
*   **`folders`:** Creates the necessary directory structure.
*   **`compose_deploy`:** Deploys the media stack using Docker Compose.
*   **`watchtower_policy`:** Configures Watchtower for automatic container updates.
*   **`backups`:** Sets up automated backups.
*   **`monitoring`:** Deploys monitoring services.
*   **`webui_integration`:** Integrates with a web UI.
*   **`mount_guard`:** A guard for mounts.
*   **`tailscale`:** Installs Tailscale.
*   **`samba_shares`:** Configures Samba shares.
*   **`app_extras`:** Installs extra applications.

## Backups and Monitoring

The backup role installs `/usr/local/bin/backup_docker.sh` and a systemd timer (`docker-backup.timer`) that archives Docker configuration and manifests under `/mnt/backups/<hostname>/YYYYMMDD`. The latest snapshot is mirrored to `/opt/mediastack/backups` for consumption by the web UI.

Key variables in `inventory/group_vars/all.yml`:

- `backups_device_uuid` – UUID of the USB drive mounted at `/mnt/backups`
- `backup_keep_daily` / `backup_keep_weekly` – retention policy
- `backup_timer_on_calendar` – systemd time specification
- `watchtower_schedule` – when Watchtower checks for image updates
- `mediastack_root` – base directory holding Docker configs and compose files

To restore a backup, copy the desired dated folder back to `/opt/mediastack` and run `docker compose up -d`.

The monitoring role exposes host metrics via `node_exporter` on port 9100 and container metrics via `cAdvisor` on port 8082.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.