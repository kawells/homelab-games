# homelab-games
Documentation for game server

## IP Addresses
eth on enp3s0 - 192.168.1.30
wlan on wlp4s0 - 192.168.1.69
tailscale - 100.114.16.12

## Setup Steps
install on SSD
adjust partitions to fill SSD

server name: games
username: ubnt
password: in BitWarden

Add microk8s, powershell, keepalived

### Install Docker
https://docs.docker.com/engine/install/ubuntu/

### Install Docker Compose
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04

### Install Tailscale
https://tailscale.com/kb/1275/install-ubuntu-2304

Test Docker Compose

### Update timezone
```
timedatectl set-timezone America/Los_Angeles
timedatectl
```

## Palworld
Server: hazy.servebeer.com:8211

Password: 8008135

If it gives you an error about not entering a password, click to join another private server (has lock icon), enter password, but hit "No" to cancel joining, then enter private server.

https://github.com/jammsen/docker-palworld-dedicated-server

On router, forward ports 8211 and 25575

```
sudo mkdir /srv
cd /srv
sudo mkdir /srv/palworld
cd /srv/palworld
sudo touch docker-compose.yml
sudo nano docker-compose.yml
sudo chown root:root /srv -R
sudo chmod g+rwx /srv -R
# paste the contents of the compose file
sudo mkdir /srv/palworld/game
sudo chmod 777 game
docker-compose pull
docker-compose up -d --remove-orphans
docker image prune -a -f
docker-compose ps
docker-compose logs -f
```

```
version: '3.9'
services:
  palworld-dedicated-server:
    build: .
    container_name: palworld-dedicated-server
    image: jammsen/palworld-dedicated-server:latest
    restart: always
    network_mode: bridge
    ports:
      - target: 8211 # gamerserver port inside of the container
        published: 8211 # gamerserver port on your host
        protocol: udp
        mode: host
      - target: 25575 # rcon port inside of the container
        published: 25575 # rcon port on your host
        protocol: tcp
        mode: host
    environment:
      - ALWAYS_UPDATE_ON_START=true
      - MAX_PLAYERS=32
      - MULTITHREAD_ENABLED=true
      - COMMUNITY_SERVER=true
      - RCON_ENABLED=true
      - RCON_PORT=25575
      - PUBLIC_IP=
      - PUBLIC_PORT=8211
      - SERVER_NAME=jank-jamerz
      - SERVER_DESCRIPTION=Palworld-Dedicated-Server running in Docker by jammsen
      - SERVER_PASSWORD=8008135
      - ADMIN_PASSWORD=ayylmao42069!
    volumes:
      - ./game:/palworld

  rcon:
    image: outdead/rcon:latest
    entrypoint: ['/rcon', '-a', 'palworld-dedicated-server:25575', '-p', 'ayylmao42069!']
    profiles: ['rcon']  # prevents booting up with `docker compose up`
```

### Crontab setup
`nano /etc/crontab`
Add: `0 3 * * * /srv/palworld/restart.sh`
```
touch /srv/palworld/restart.sh
nano /srv/palworld/restart.sh
```
restart.sh:
```
#!/bin/bash

# Change to the directory containing the docker-compose.yml file
cd /srv/palworld/

# Pull new images
docker-compose pull

# Restart all containers using Docker Compose
docker-compose restart
```

Make executable
`chmod +x /srv/palworld/restart.sh`
Run update/restart script
`bash /srv/palworld/restart.sh`

## Satisfactory
[Docker Setup](https://hub.docker.com/r/wolveix/satisfactory-server)
```
sudo mkdir /srv/satisfactory
cd /srv/satisfactory
sudo mkdir /srv/satisfactory/game
sudo chmod 777 /srv/satisfactory/game
sudo touch docker-compose.yml
sudo nano docker-compose.yml
# paste the contents of the compose file

```
