# Mattermost on Openstack

This doc shows some things required on Openstack for this to work.

## Infrastructure
Given: OpenStack Project by someone


Create Security Group `mattermost`
- Incoming tcp/22 from 0.0.0.0 allowed
- Incoming tcp/80 from 0.0.0.0 allowed
- Incoming tcp/443 from 0.0.0.0 allowed
- Outgoing ipv4/ipv6 traffic to 0.0.0.0 allowed

Create Floating IP for later use. 

Create an Openstack Swift Container called `mattermost_backup` will serve for Backups

DNS was setup as mattermost.technat.dev and *.mattermost.technat.dev which are pointing to the Floating IP 

Create a server with the following params:
OS: Ubuntu 24.04
Disk: 40GB OS Disk
Size: m1.medium
Networks: private (floating IP is allocated later)
Security Groups: mattermost
Key Pairs: not used because they cannot import ed25519 keys, cloud-init will be used
Cloud-Init Config: [./cloud-init.yaml](./cloud-init.yaml)

Floating IP was assigned after instance has been launched. An additional 50GB volume was attached as well from the Openstack console to this instance.

## OS Configurations

### Mount volume 

Check the addtional volume is mounted on the instance. Use `lsblk` for that. For the following commands this will be `/dev/vdb`.

Partition and format the disk using fdisk: `fdisk /dev/vdb`, then type:

```
g
n
enter
enter
enter
w
```

Next create a filesystem on the newly created partition using `mkfs.ext4 /dev/vdb1`.

If you don't have existing data in `/var/lib/docker/volumes` you can proceed with adding the fstab entry. Otherwise mount the partition in a temporary location using `mount /dev/vdb1 /mnt`.

Then stop docker and it's containers: `docker compose down && sudo systemctl stop docker`.
Next copy all files from `/var/lib/docker/volumes` to `/mnt` using rsync: `rsync -axHAWXS --numeric-ids --info=progress2 /var/lib/docker/volumes/ /mnt`.

The options explained (taken from https://www.baeldung.com/linux/rsync-clone-file-system-hierarchy):
-a is the archive mode: it recurses into directories, copies symlinks as symlinks, and preserves permissions, owner, group, modification times, device files, and special files.
-x doesn’t cross filesystem boundaries.
-H preserves hard links.
-A preserves ACLs.
-W disables the delta-transfer algorithm used to reduce network usage. It’s a convenient way to boost speed when both the source and destination are local paths.
-X updates the destination extended attributes to be the same as the source ones.
-S tries to handle sparse files efficiently so that they take up less space on the destination.
–numeric-ids uses numeric IDs instead of trying to map them. It’s notably needed for backups of jailed systems (BSD jails, OpenVZ, VServer, LXC) that appear to have bogus IDs when seen from their host system because they have their own ID maps.
–info=progress2 outputs statistics based on the whole transfer, rather than individual files.

Now that the data is copied over, you can unmount the volume in /mnt: `sudo umount /dev/vdb1`

And create the fstab-entry using `sudo vim /etc/fstab`. Add a line that looks like this:
```
/dev/vdb1 /var/lib/docker/volumes ext4 defaults 0 1
```

And apply this using `sudo mount -a` and `sudo systemctl daemon-reload`. Now you should see the volumes as before (if there were some) using `docker volume ls`. Start the containers again.

#### Docker
Was installed according to their Instructions: https://docs.docker.com/engine/install/ubuntu/
insy is a member of the docker group: `sudo usermod -aG docker insy`
Docker Compose was installed as port of Docker using the docker-compose-plugin package

