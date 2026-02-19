# server-stack
[![Build Status](https://circleci.com/gh/pedbarbosa/server-stack.svg?style=shield)](https://app.circleci.com/pipelines/github/pedbarbosa/server-stack)

## Service list

This Docker Compose stack uses the following web services:
  - [HomeAssistant](https://hub.docker.com/r/homeassistant/home-assistant) - home automation
  - [Immich](https://ghcr.io/immich-app/immich-server) - photo archive
  - [Jellyfin](https://hub.docker.com/r/linuxserver/jellyfin) - private media streaming
  - [Lidarr](https://hub.docker.com/r/blampe/lidarr) - music management
  - [ProjectSend](https://hub.docker.com/r/linuxserver/projectsend) - private file sharing
  - [Radarr](https://hub.docker.com/r/linuxserver/radarr) - movie management 
  - [RuTorrent](https://hub.docker.com/r/crazymax/rtorrent-rutorrent) - torrent handling (backed by a [gluetun](https://hub.docker.com/r/qmcgaw/gluetun) VPN client)
  - [Seer](https://ghcr.io/seerr-team/seerr) - Movie and TV show request handling
  - [Sickchill](https://hub.docker.com/r/linuxserver/sickchill) - TV shows management
  - [Sonarr](https://hub.docker.com/r/linuxserver/radarr) - TV shows management 
  - [Syncthing](https://hub.docker.com/r/linuxserver/syncthing) - private backups
  - [Vaultwarden](https://hub.docker.com/r/vaultwarden/server) - password management

as well as the following support services:
  - [Autoheal](https://hub.docker.com/r/willfarrell/autoheal) - container monitoring
  - [Caddy](https://hub.docker.com/r/pedbarbosa/caddy) - web proxy and SSL certificate handler
  - [Cloudflared](https://hub.docker.com/r/cloudflare/cloudflared) - secure tunnel to Cloudflare
  - [Collectd Graph Panel](https://hub.docker.com/r/pedbarbosa/cgp) - collectd statistics viewer
  - [DDclient](https://hub.docker.com/r/linuxserver/ddclient) - dynamic DNS updates
  - [KODI](https://hub.docker.com/r/matthuisman/kodi-headless) - headless KODI server
  - [LibreSpeed](https://lscr.io/linuxserver/librespeed) - private speed testing
  - [MariaDB](https://hub.docker.com/r/linuxserver/mariadb) - SQL server
  - [Mosquitto](https://hub.docker.com/_/eclipse-mosquitto) - MQTT server
  - [Postfix](https://hub.docker.com/r/pedbarbosa/postfix) - mail server
  - [Prowlarr](https://hub.docker.com/r/linuxserver/prowlarr) - index manager for "arr" apps
  - [Smokeping](https://hub.docker.com/r/linuxserver/smokeping) - internet latency statistics
  - [Sungather](https://hub.docker.com/r/pedbarbosa/sungather) - solar panel inverter metrics
  - [Zigbee2MQTT](https://hub.docker.com/r/koenkk/zigbee2mqtt) - Zigbee to MQTT bridge

## Usage

To use the stack, make sure to create a .env file, and populate it with the required variables:

```
cp .env_example .env
vim .env
```

### Start all services
```bash
docker compose up -d
```

### Start specific service groups using profiles
```bash
# Media services only
docker compose --profile media up -d

# Home automation stack
docker compose --profile home-automation up -d

# Networking services
docker compose --profile networking up -d

# Multiple profiles
docker compose --profile media --profile monitoring up -d
```

### Available Profiles

- `home-automation` - IoT and smart home services
- `immich` - Photo management with AI features
- `media` - Media management and streaming
- `monitoring` - Health and performance monitoring
- `networking` - VPN, proxy, SSL services
- `utilities` - Database and utility services

## Tools

### Container security patcher

To update all containers' packages to their latest versions, run the script below. Please note that ALL containers running on the server will have their packages upgraded!

```
./patch_containers
```

To run it periodically, run the following to add it to your crontab:

```
crontab -l | { cat; echo -e "# Container patcher\n0 5 * * * $pwd/patch_containers"; } | crontab -
```

### Network devices certificate update

Check if your network devices need a certificate update by running:

```
./update_certs -v
```

To run it periodically, run the following to add it to your crontab:

```
crontab -l | { cat; echo -e "# Certificate checker\n0 6 * * * $pwd/update_certs"; } | crontab -
```

### Jellyfin with nvenc transcoding

To use nvenc transcoding in Jellyfin, on an Ubuntu host run the following:

Set up the libnvidia-container repository:

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Install the Nvidia container toolkit and configure it:

```
apt update
apt install nvidia-container-toolkit

nvidia-ctk runtime configure
systemctl restart docker
```

If not available or not required, remove the 'runtime' and the 'NVIDIA_VISIBLE_DEVICES' environment variable from the 'jellyfin' container
