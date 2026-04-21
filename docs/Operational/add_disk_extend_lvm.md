# Add and Extend Disk with LVM in Linux

This guide explains how to add a new disk to a Linux system, format it, join it to an existing Volume Group (VG), and extend the Logical Volume (LV) accordingly.

---

## Step 1: View Available Disks

```bash
lsblk
```

> This command lists all block devices and helps you identify the new disk (e.g., `/dev/vdb`).

---

## Step 2: Partition the New Disk

```bash
fdisk /dev/vdb
```

Inside `fdisk`, perform the following actions:

1. `g` — Create a new empty GPT partition table
2. `n` — Create a new partition (accept defaults)
3. `t` — Change the partition type (default is fine for LVM)
4. `w` — Write changes and exit

This will create `/dev/vdb1`.

---

## Step 3: Extend the Volume Group

```bash
vgextend ubuntu-vg /dev/vdb1
```

> This adds the new partition to the `ubuntu-vg` Volume Group.

---

## Step 4: Verify VG and LV

```bash
vgs
lvs
```

> These commands show the current state of Volume Groups and Logical Volumes.

---

## Step 5: Extend the Logical Volume

```bash
lvextend /dev/ubuntu-vg/ubuntu-lv /dev/vdb1
```

> This extends the LV by using the space from the new partition.

---

## Step 6: Resize the Filesystem

```bash
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

> This resizes the filesystem to use the new space. This assumes your root filesystem uses `ext4`.

---

Done! You’ve successfully added a new disk and extended your LVM volume.
