# Mattermost on Openstack

This doc explains how to setup and prepare an Openstack Project for a Mattermost instance hosted with docker compose.

## Infrastructure
Given: OpenStack Project by someone

Create Security Group `mattermost`
- Incoming tcp/22 from 0.0.0.0 allowed
- Incoming tcp/80 from 0.0.0.0 allowed
- Incoming tcp/443 from 0.0.0.0 allowed
- Outgoing ipv4/ipv6 traffic to 0.0.0.0 allowed

Create Floating IP for later use named `mattermost`. Ensure you have a DNS record pointing to this domain that will be used later.

Create an Openstack Swift Container called `mattermost_backup` for Backups.

Create a server with the following params:
- **OS**: Ubuntu 24.04
- **Disk**: 40GB OS Disk
- **Size**: m1.medium
- **Networks**: private (floating IP is allocated later)
- **Security Groups**: mattermost
- **Key Pairs**: not used because they cannot import ed25519 keys, cloud-init will be used
- **Cloud-Init Config**: [./cloud-init.yaml](./cloud-init.yaml)

The Floating IP was assigned to the instance after it has been launched. An additional 50GB volume was attached as well from the Openstack console to this instance that will be used for Docker volumes later.

## OS Configurations

Some things to setup on the Linux box. SSH access and hardening should already be done by cloud-init so you should be able to just login to the box via it's floating IP (or DNS name).

### Configure external volume

Check the additional volume is mounted on the instance. Use `lsblk` for that. Since this is a cloud VM you should see two `vdb*` devices. For the following commands `/dev/vdb` will be the second volume used for Docker storage.

Partition and format the disk using fdisk: `fdisk /dev/vdb`, then type:

```
g
n
enter
enter
enter
w
```

This will partiton the disk using GPT and create one partition using the full size.

Next create a filesystem on the newly created partition using `mkfs.ext4 /dev/vdb1`.

If you don't have existing data in `/var/lib/docker/volumes` you can jump to adding the fstab entry at the bottom of this section. Otherwise mount the partition in a temporary location using `mount /dev/vdb1 /mnt`. Ensure `/mnt` exists and can be used (e.g there's no other mount/data stored).

Then stop docker and it's containers: `docker compose down && sudo systemctl stop docker`.

Next copy all files from `/var/lib/docker/volumes` to `/mnt` using rsync: `rsync -axHAWXS --numeric-ids --info=progress2 /var/lib/docker/volumes/ /mnt`.

The options for this rsync command explained (taken from https://www.baeldung.com/linux/rsync-clone-file-system-hierarchy):
-a is the archive mode: it recurses into directories, copies symlinks as symlinks, and preserves permissions, owner, group, modification times, device files, and special files.
-x doesn’t cross filesystem boundaries.
-H preserves hard links.
-A preserves ACLs.
-W disables the delta-transfer algorithm used to reduce network usage. It’s a convenient way to boost speed when both the source and destination are local paths.
-X updates the destination extended attributes to be the same as the source ones.
-S tries to handle sparse files efficiently so that they take up less space on the destination.
–numeric-ids uses numeric IDs instead of trying to map them. It’s notably needed for backups of jailed systems (BSD jails, OpenVZ, VServer, LXC) that appear to have bogus IDs when seen from their host system because they have their own ID maps.
–info=progress2 outputs statistics based on the whole transfer, rather than individual files.

Now that the data is copied over, you can unmount the volume in `/mnt`: `sudo umount /dev/vdb1`

And create the fstab-entry using `sudo vim /etc/fstab`. Add a line that looks like this:
```
/dev/vdb1 /var/lib/docker/volumes ext4 defaults 0 1
```

Note: if you want a more reliable mount use `blkid /dev/vdb1 | cut -d ' ' -f 2 ` and use the UUID instead of `/dev/vdb1` in the mount-entry.

Finally apply this new mount-entry using `sudo mount -a` and `sudo systemctl daemon-reload`. It's also a good idea to reboot the instance and see if the volume is still mounted where you expect it to be.

#### Docker

Now that there is a volume for docker volumes let's install docker. Use their Instructions for Ubuntu: https://docs.docker.com/engine/install/ubuntu/

Make sure `insy` is a member of the docker group: `sudo usermod -aG docker insy`

Docker Compose was installed as port of Docker using the docker-compose-plugin package

Now you can use the configs from this repo to setup Mattermost.