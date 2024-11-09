---
title: NAS (Network Attached Storage)
---

# System Setup Summary

## External USB HDD
Connect an external USB HDD to the system and identify it using the `lsblk` command. After confirming the device name, create a mount point at `/mnt/external`. To ensure the drive auto-mounts at boot, retrieve the UUID of the device using `blkid`:

```bash
sudo blkid /dev/sdd1
```

Add an entry to `/etc/fstab` to automatically mount the drive on startup:

```bash
UUID=your-uuid-here /mnt/external ext4 defaults 0 2
```

Reload the system with `sudo mount -a` to mount the drive immediately.

## Western Digital NAS Drives (RAID1 Removal)
Identify two Western Digital NAS drives, originally configured in RAID1, in the system. These drives were previously used in a QNAP NAS, which typically uses a RAID configuration with LVM for disk management. Use this configuration to combine RAID for redundancy and LVM for flexible volume management.

Identify the active RAID arrays using the following command:

```bash
cat /proc/mdstat
```

To remove the RAID1 setup, stop the RAID arrays using `mdadm`:

```bash
sudo mdadm --stop /dev/md123
sudo mdadm --stop /dev/md124
sudo mdadm --stop /dev/md125
sudo mdadm --stop /dev/md126
sudo mdadm --stop /dev/md127
```

Clear the RAID superblocks from the drives:

```bash
sudo mdadm --zero-superblock /dev/sdb3
sudo mdadm --zero-superblock /dev/sdc3
```

Deactivate the logical volumes:

```bash
sudo lvchange -an vg1/lv1
sudo lvchange -an vg1/lv544
```

Remove the logical volumes and volume groups:

```bash
sudo lvremove /dev/vg1/lv1
sudo lvremove /dev/vg1/lv544
sudo vgremove vg1
```

Clear the physical volumes:

```bash
sudo pvremove /dev/sdb3
sudo pvremove /dev/sdc3
```

Delete existing partitions on `/dev/sdb` and `/dev/sdc` using `fdisk`:

```bash
sudo fdisk /dev/sdb
sudo fdisk /dev/sdc
```

Create new ext4 filesystems on both drives:

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdc1
```

Create mount points at `/mnt/drive1` and `/mnt/drive2`, and mount the drives:

```bash
sudo mkdir -p /mnt/drive1
sudo mkdir -p /mnt/drive2
sudo mount /dev/sdb1 /mnt/drive1
sudo mount /dev/sdc1 /mnt/drive2
```

Retrieve the UUIDs of both drives:

```bash
sudo blkid /dev/sdb1
sudo blkid /dev/sdc1
```

Add entries to `/etc/fstab` to ensure auto-mounting:

```bash
# Western Digital 4TB #1
/dev/disk/by-uuid/5feead0e-89c5-4339-a777-fcbfa9ace876 /mnt/drive1 ext4 defaults 0 1

# Western Digital 4TB #2
/dev/disk/by-uuid/9b73c31c-e610-4f56-8e3f-7362f61904fa /mnt/drive2 ext4 defaults 0 1
```

## Seagate 14TB Drive
Unmount the Seagate 14TB drive, which was previously mounted at `/seagate`:

```bash
sudo umount /seagate
```

Retrieve the UUID of the drive:

```bash
sudo blkid /dev/sdd1
```

Add an entry to `/etc/fstab` to mount the drive at `/mnt/drive3`:

```bash
# Seagate External 14TB
/dev/disk/by-uuid/b40810fe-764a-428a-8f0c-663db4745966 /mnt/drive3 ext4 defaults 0 1
```

Reload the system to apply the changes:

```bash
sudo mount -a
```

## Final `/etc/fstab` Configuration:
```bash
# Western Digital 4TB #1
/dev/disk/by-uuid/5feead0e-89c5-4339-a777-fcbfa9ace876 /mnt/drive1 ext4 defaults 0 1

# Western Digital 4TB #2
/dev/disk/by-uuid/9b73c31c-e610-4f56-8e3f-7362f61904fa /mnt/drive2 ext4 defaults 0 1

# Seagate External 14TB
/dev/disk/by-uuid/b40810fe-764a-428a-8f0c-663db4745966 /mnt/drive3 ext4 defaults 0 1
```

## Installing Samba

To install Samba on the system, run the following command:

```bash
sudo apt install samba
```

Once installed, start and enable the Samba service:

```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

Verify that the Samba service is running:

```bash
sudo systemctl status smbd
```

## Samba Configuration

To configure the `Public` share, update your `/etc/samba/smb.conf` file with the following:

```ini
[Public]
   path = /mnt/drive3
   browseable = yes
   read only = yes
   guest ok = yes
   write list = kyle, liz
```

Restart the Samba service to apply the changes:

```bash
sudo systemctl restart smbd
```

## Adding the `liz` User to the System

To add the `liz` user to the system, use the following command:

```bash
sudo adduser liz
```

Follow the prompts to create the user and set a password.

## Adding Users to Samba

To add the `kyle` and `liz` users to Samba, use the following commands:

```bash
sudo smbpasswd -a kyle
sudo smbpasswd -a liz
```

This will prompt you to set Samba-specific passwords for both users.

## Testing the Samba Configuration with `smbclient`

To test the Samba configuration from a client machine, use the `smbclient` tool to connect to the share. For example, to test the `Public` share:

For the `kyle` user:

```bash
smbclient //nas.lan/Public -U kyle
```

For the `liz` user:

```bash
smbclient //nas.lan/Public -U liz
```

For guest access (if allowed):

```bash
smbclient //nas.lan/Public -N
```

Use the `ls` command within `smbclient` to list the contents of the share:

```bash
ls
```

## Unmount the Drives
Ensure the drives are unmounted before proceeding.

```
sudo umount /mnt/drive1
sudo umount /mnt/drive2
```

## Scan for Bad Sectors Using `badblocks`
Use `badblocks` to scan for bad sectors on each drive. This will perform a non-destructive read-only test.

```
sudo badblocks -sv /dev/sdb
sudo badblocks -sv /dev/sdc
```

If you'd like a more thorough test (which will take longer), you can use the following command, which writes test patterns to the disk and reads them back:

```
sudo badblocks -wsv /dev/sdb
sudo badblocks -wsv /dev/sdc
```

**Warning**: Using the `-w` option will destroy all data on the drives.

## Install ZFS on Ubuntu Server 24.04
Install the ZFS utilities if they are not already installed:

```
sudo apt update
sudo apt install zfsutils-linux
```

## Destroy the Current ext4 Filesystem (if necessary)
After scanning for bad sectors, if you are certain you want to proceed, you’ll need to wipe the existing ext4 partitions on both drives:

```
sudo wipefs -a /dev/sdb
sudo wipefs -a /dev/sdc
```

## Create a New ZFS Pool
Once the partitions are wiped, create a ZFS pool. Replace `media` with your desired pool name.

- For a mirrored setup (provides redundancy):

```
sudo zpool create media mirror /dev/sdb /dev/sdc
```

- For a striped setup (combines space without redundancy):

```
sudo zpool create media /dev/sdb /dev/sdc
```

## Check the Status of the ZFS Pool
Verify the status of the new ZFS pool:

```
sudo zpool status
```

## Set Auto-Mount at Boot (Optional)
Ensure the ZFS pool mounts automatically at boot:

```
sudo zpool set autoexpand=on media
```

## Plex

Install Plex on the server by [downloading the deb](https://www.plex.tv/media-server-downloads/?cat=computer&plat=linux) or by running:

```shell
curl -L https://downloads.plex.tv/plex-media-server-new/1.40.5.8921-836b34c27/debian/plexmediaserver_1.40.5.8921-836b34c27_amd64.deb | sudo dpkg -i -
```

Visit https://www.plex.tv/claim/, log in, and then copy the claim code.

Claim your Plex instance using the claim code you generated earlier by running the following command on the server:

```shell
curl -X POST 'http://127.0.0.1:32400/myplex/claim?token=CLAIM_CODE' 
```

This request may take a while to complete.

### WTF

On the server, stop Plex:

```shell
sudo systemctl stop plexmediaserver
```

Edit the Plex configuration file:

```shell
sudo nano /var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server/Preferences.xml
```

And remove the values from the following attributes, if they exist:

* `PlexOnlineName`
* `PlexOnlineToken`
* `PlexOnlineEmail`
* `PlexOnlineHome` (if it exists)

Start Plex:

```shell
sudo systemctl start plexmediaserver
```

Try again.
