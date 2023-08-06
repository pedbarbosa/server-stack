# server-stack
[![Build Status](https://circleci.com/gh/pedbarbosa/server-stack.svg?style=shield)](https://app.circleci.com/pipelines/github/pedbarbosa/server-stack)

This Docker Compose stack includes images for the following web services:
  - [HomeAssistant](https://hub.docker.com/r/homeassistant/home-assistant) - home automation
  - [Jellyfin](https://hub.docker.com/r/linuxserver/jellyfin) - private media streaming
  - [ProjectSend](https://hub.docker.com/r/linuxserver/projectsend) - private file sharing
  - [RuTorrent](https://hub.docker.com/r/crazymax/rtorrent-rutorrent) - torrent handling (backed by a [NordVPN](https://hub.docker.com/r/bubuntux/nordlynx) VPN client)
  - [Sickchill](https://hub.docker.com/r/linuxserver/sickchill) - TV shows management
  - [Syncthing](https://hub.docker.com/r/linuxserver/syncthing) - private backups
  - [Vaultwarden](https://hub.docker.com/r/vaultwarden/server) - password management

as well as the following images for support services:
  - [Autoheal](https://hub.docker.com/r/willfarrell/autoheal) - container monitoring
  - [Collectd Graph Panel](https://hub.docker.com/r/pedbarbosa/docker-cgp) - collectd statistics viewer
  - [DDclient](https://hub.docker.com/r/linuxserver/ddclient) - dynamic DNS updates
  - [MariaDB](https://hub.docker.com/r/linuxserver/mariadb) - SQL server
  - [Nginx](https://hub.docker.com/_/nginx) - proxy for web services
  - [Postfix](https://hub.docker.com/r/pedbarbosa/postfix) - mail server
  - [Smokeping](https://hub.docker.com/r/linuxserver/smokeping) - network statistics
  - [Swag](https://hub.docker.com/r/linuxserver/swag) - SSL certificate handler

To use the stack, make sure to create a .env file with the required variables:

```
cp .env_example .env
vim .env
```

and then launch it using:

```
docker compose up -d
```

### Container security patcher

To update all containers' packages to their latest versions, run the script below. Please note that ALL containers running on the server will have their packages upgraded!

```
./patch_containers
```

To run it periodically, run the following to add it to your crontab:

```
crontab -l | { cat; echo -e "# Container patcher\n0 5 * * * $pwd/patch_containers"; } | crontab -
```

### Certificate checker for Swag

To ensure the Nginx proxy is using the latest Swag certificate, run the following:

```
DOMAIN=<your_domain_name>
./update_certs
```

To run it periodically, run the following to add it to your crontab:

```
DOMAIN=<your_domain_name>
crontab -l | { cat; echo -e "# Certificate checker\n0 6 * * * DOMAIN=$DOMAIN $pwd/update_certs"; } | crontab -
```

### nvenc for jellyfin

To use nvenc transcoding in jellyfin, on an Ubuntu host run the following:

Set up the nvidia-docker repository:

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu22.04/nvidia-docker.list > /etc/apt/sources.list.d/nvidia-docker.list
```

Install the Nvidia container toolkit and configure it:

```
apt update
apt install nvidia-container-toolkit

nvidia-ctk runtime configure
systemctl restart docker
```

If not available or not required, remove the 'runtime' and the 'NVIDIA_VISIBLE_DEVICES' environment variable from the 'jellyfin' container

