# An example of Raspberry Pi NAS/Samba/UPnP/Docker-multipurpose server provisioned with Ansible

This is an example Ansible configuration for provisioning Raspberry Pi OS.
It's based on my own Ansible setup for my Raspberry Pi 4 with Raspberry Pi OS Lite 64 bit.
It should work for other Debian based distros as well.

## Why?

Over six years ago I purchased my Pi 4 and setupped it.
At that time the OS for it was 32 bit Raspbian which was based on Debian 10 IIRC.
Years went by, 64 bit support arrived and support for Raspbian ended. Bit by bit my setup became like spilt milk.
I knew I should update the OS but was frustrated how to setup everything I had in my current setup.
So I went through all I had, got rid off some unnecessary stuff and tidied the rest a lot.
I decided to solve the setup and configuration for good and created Ansible playbook for provisioning my little server.

## What?

The Ansible configuration is structuded as such:

* `group_vars/`:
    * `main.yml`: Global role independent variables
    * `vault.yml`: Global role independent vault
* `inventory/`:
    * `hosts.yml`: Hosts configuration
* `roles/`:
    * `assetupnp/`: Configuration for [Asset UPnP DLNA compatible media server](https://www.dbpoweramp.com/asset-upnp-dlna.htm)
        * `files/`:
            * `.dBpoweramp/`: Place here your instance / media library files
            * `bin/`: Place here binaries
        * `tasks/main.yml`: Tasks for setupping
        * `vars/main.yml`: Variables used by this role
    * `code/`: Configuration for custom code to be executed in the host
        * `handlers/main.yml`: Handlers for restarting services setupped with this role
        * `tasks/`:
            * `dy-fi-ip-updater.yml`: Tasks for setupping [Dy.fi IP updater](https://github.com/maszaa/dy-fi-ip-updater)
            * `heos-scrobbler.yml`: Tasks for setupping [HEOS Scrobbler](https://github.com/maszaa/heos-scrobbler)
            * `main.yml`: Other custom code related tasks, also imports Dy.fi IP updater and HEOS Scrobbler tasks
        * `templates/`:
            * `scripts/`: Various bash and Python scripts for backup and such
            * `systemd/`: Systemd service unit files for Dy.fi IP updater and HEOS Scrobbler
            * `toml/heos-scrobbler.secrets.toml.j2`: Secret file for HEOS Scrobbler
    * `cron/`: Configuration for cron tasks
        * `tasks/main.yml`: Tasks for creating the cronjobs
        * `vars/main.yml`: Cronjob definitions
    * `dependencies/`: Configuration for installing required dependencies to the host
        * `files/`: Shell scripts for installing [uv](https://docs.astral.sh/uv/) and rootless Docker, Docker daemon config as json
        * `handlers/main.yml`: Handler for restarting rootless Docker
        * `tasks/`:
            * `docker.yml`: Tasks for setupping rootless Docker
            * `main.yml`: Tasks for setupping other dependencies, also imports Docker tasks
    * `homeassistant/`: Configuration for initializing [Homeassistant](https://www.home-assistant.io/) as a Docker container
        * `tasks/main.yml`: Tasks for setupping
        * `templates/docker-compose.yml.j2`: Docker Compose configuration for the container
            * I couldn't get `network_mode: host` to work so mDNS unfortunately doesn't work for me
        * `vars/main.yml`: Variables for this role
    * `mail/`: Tasks for adding mail sending capabilities to the host
        * `tasks/main.yml`: Tasks for setupping mail configuration
        * `templates/`: msmtp and mail aliases configuration
    * `minidlna/`: Tasks for setupping [minidlna](https://wiki.archlinux.org/title/ReadyMedia) media server
        * `tasks/main.yml`: Tasks for setupping
        * `templates/minidlna.conf.j2`: minidlna configuration file
    * `mounts/`: Configuration for additional mounts to place to `/etc/fstab`
        * `tasks/main.yml`: Tasks for setupping
        * `vars/main.yml`: Configuration for `/etc/fstab` mounts (items)
    * `rclone/`: Configuration for [rclone](https://rclone.org/)
        * `tasks/main.yml`: Tasks for setupping
        * `templates/rclone.conf.j2`: Configuration file for rclone
        * `vars/main.yml`: Variables used in rclone configuration file
    * `samba`: Tasks for setupping Samba/SMB network shares
        * `tasks/main.yml`: Tasks for setupping
        * `templates/smb.conf.j2`: Configuration file for Samba server
    * `ssh/`: Tasks for setupping SSH-ing ability to the host
        * `files/`: `authorized_keys` and `known_hosts` files
        * `tasks/main.yml`: Tasks for setupping
    * `system/`: Tasks for various system configuration tasks like increasing inotify and configuring logrotate for logs of unprivileged user (pi)
        * `tasks/main.yml`: Tasks for setupping
        * `templates/logrotate.pi.j2`: Configuration file for logrotate
    * `update/tasks/main.yml`: Tasks for updating system dependencies
    * `site.yml`: The playbook. Roles are tagged so you can choose what to execute.

## How?

You need Python 3 installed in a Linux OS (WSL works as well).

Create Python virtual environment and install Ansible with `pip install -r requirements.txt`.

**DISCLAIMER**: This playbook doesn't work out of box as-is. You need to create the vault and populate it with required variables.
You also need to check `group_vars\all\main.yml` and `inventory\hosts.yml` and tune configuration files of roles to suit your setup.

### Caveats

* If you are going to use rclone, you may need to tune its setup, I copied the token from my previous setup
    * See https://rclone.org/remote_setup/ and https://rclone.org/drive/
* If you are going to use a Gmail account for sending email,
see https://arnaudr.io/2020/08/24/send-emails-from-your-terminal-with-msmtp/ for instructions how to setup app password for an account
* If you are going to use Dy.fi IP updater or/and HEOS Scrobbler,
see https://github.com/maszaa/dy-fi-ip-updater or/and https://github.com/maszaa/heos-scrobbler for instructions
* If you are going to use Healthchecks.io for monitoring cronjobs, see https://healthchecks.io/docs/
