# Mattermost on Openstack

## Infrastructure
Given: OpenStack Project
Security Group: mattermost
- Incoming tcp/22 from 0.0.0.0 allowed
- Incoming tcp/80 from 0.0.0.0 allowed
- Incoming tcp/443 from 0.0.0.0 allowed
- Outgoing ipv4/ipv6 traffic to 0.0.0.0 allowed

Floating IP: 86.119.45.128

Storage: Openstack Swift Container called mattermost_backup will serve for Backups

DNS: mattermost.technat.dev & *.mattermost.technat.dev are pointing to Floating IP from external DNS hoster

Server: one linux box:
OS: Ubuntu 24.04
Disk: 40GB OS Disk
Size: m1.medium
Networks: private (floating IP is allocated later)
Security Groups: mattermost
Key Pairs: not used because they cannot import ed25519 keys, cloud-init will be used
Cloud-Init Config: [./cloud-init.yaml](./cloud-init.yaml)

Floating IP was assigned after instance has been launched

## OS Configurations

### SSH Hardening
These directives have changed in /etc/ssh/sshd_config:
```
PubKeyAuthentication yes
PermitRootLogin no
PasswordAuthentication no
X11Forwarding no
```
A restart of the service was triggered afterwards using `sudo systemctl restart ssh`.

### Docker
Was installed according to their Instructions: https://docs.docker.com/engine/install/ubuntu/
insy is a member of the docker group: `sudo usermod -aG docker insy`
Docker Compose was installed as port of Docker using the docker-compose-plugin package

### Application Configuraiton

Docker-Compose file: [./docker-compose.yml](./docker-compose.yml)

Start using `docker compose up -d`

### Backup
TODO: am einfachsten Openstack Swift aber das ist nicht S3 kompatibel und somit dürfte es
schwierig sein ein Projekt zu finden das Swift kann und gleichzetig einfach und schnell Container volumes/dbs backupen kann

