
# Add and Mount a Non-LVM Partition to /mnt

This guide explains how to create a new partition on a disk, format it, mount it to `/mnt`, and make the mount persistent using `/etc/fstab`.

---

## Step 1: Partition the Disk

```bash
fdisk /dev/vdb
```

Inside `fdisk`, perform the following actions:

1. `g` — Create a new empty GPT partition table
2. `n` — Create a new partition (accept defaults)
3. `t` — Change the partition type (default is fine for ext4)
4. `w` — Write changes and exit

This will create `/dev/vdb2`.

---

## Step 2: Format the New Partition

```bash
mkfs.ext4 /dev/vdb2
```

> Formats the partition with the ext4 filesystem.

---

## Step 3: Mount the Partition

```bash
mount /dev/vdb2 /mnt
```

> Temporarily mounts the new partition to `/mnt`.

---

## Step 4: Make the Mount Persistent

First, get the UUID of the partition:

```bash
blkid /dev/vdb2
```

Add a line to `/etc/fstab`:

```
UUID=xxxx-xxxx   /mnt   ext4   defaults   0   2
```

Replace `xxxx-xxxx` with the actual UUID found via `blkid`.

---

## Step 5: Test the fstab Entry

```bash
mount -a
```

> This will re-mount everything listed in `/etc/fstab` to confirm your configuration works.

---

Done! The new partition is now mounted to `/mnt` and will persist across reboots.
