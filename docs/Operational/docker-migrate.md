# Docker and Containerd Storage Migration to LVM

## Problem Statement

The root disk (20GB) was filling up because Docker images, containers, and volumes were stored on the root partition. The server had an external 20GB disk (vdb) configured as an LVM logical volume. The goal was to move all Docker and containerd data to the LVM volume to prevent the root disk from filling up.

## Why This Was Necessary

Docker uses two separate storage locations:
- /var/lib/docker - Docker metadata, container configurations, and volumes
- /var/lib/containerd - Container image layers, snapshots, and overlay filesystem data

Changing only Docker's data-root setting does not move containerd data. Containerd continued writing image layers to /var/lib/containerd on the root disk, causing disk usage to increase even after Docker was reconfigured.

## System Configuration

Root disk: /dev/vda1 mounted at /
External disk: /dev/vdb with LVM volume yellowstone-lv_docker mounted at /mnt/docker-data

## Step-by-Step Commands and Explanations

### 1. Initial Assessment

Check current disk usage:

df -h

Purpose: Identify which filesystems are reaching capacity. Shows used space, available space, and mount points for all filesystems.

Check block device layout:

lsblk

Purpose: Display all block devices, partition layouts, LVM volumes, and current mount points. Confirms that LVM volume is available and correctly sized.

Check Docker's current storage location:

docker info | grep "Docker Root Dir"

Purpose: Verify where Docker is configured to store its data. Should show /var/lib/docker by default.

### 2. Configure Docker to Use LVM Volume

Stop Docker daemon and related services:

sudo systemctl stop docker docker.socket containerd

Purpose: Ensure no write operations occur during migration. Stopping both Docker and containerd prevents data corruption.

Create or edit Docker daemon configuration:

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "data-root": "/mnt/docker-data"
}
EOF
```

Purpose: Set Docker's data root directory to the LVM mount point. This tells Docker to store all future data on the external disk.

### 3. Move Containerd Data to LVM

The critical step that was initially missed. Containerd stores image layers and snapshots separately from Docker.

Stop containerd service:

sudo systemctl stop containerd

Purpose: Prevent containerd from writing to disk during data migration.

Create target directory on LVM:

sudo mkdir -p /mnt/docker-data/containerd

Purpose: Create the directory structure where containerd data will be stored on the LVM volume.

Move containerd data to LVM:

```
sudo rsync -avxP /var/lib/containerd/ /mnt/docker-data/containerd/
```

Purpose: Copy all containerd data including image layers, snapshots, and metadata to the LVM volume. The flags preserve permissions, recursively copy directories, and show progress.

Rename old containerd directory as backup:

sudo mv /var/lib/containerd /var/lib/containerd.bak

Purpose: Keep a backup in case the migration fails. This allows rollback without data loss.

Generate default containerd configuration:

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Purpose: Create a default configuration file for containerd if one does not exist. This file controls containerd's behavior including storage paths.

Edit containerd configuration to use LVM path:

sudo sed -i 's|root = ".*"|root = "/mnt/docker-data/containerd"|' /etc/containerd/config.toml

Purpose: Change the root directory setting in containerd config from default /var/lib/containerd to the LVM location.

Alternative manual edit if sed command fails:

sudo nano /etc/containerd/config.toml

Find the line starting with root = and change it to:
root = "/mnt/docker-data/containerd"

### 4. Restart Services

Start containerd with new configuration:

sudo systemctl start containerd

Purpose: Launch containerd using the updated config pointing to LVM storage.

Start Docker daemon:

sudo systemctl start docker

Purpose: Launch Docker which will now use /mnt/docker-data as its data root.

### 5. Verify Migration Success

Check Docker's current storage location:

docker info | grep "Docker Root Dir"

Expected output: Docker Root Dir: /mnt/docker-data

Check that containerd overlays are using LVM path:

mount | grep overlay | head -5

Purpose: Examine active overlay mounts. All lowerdir and upperdir paths should show /mnt/docker-data/containerd/... not /var/lib/containerd/...

Check that no containerd paths remain on root disk:

mount | grep overlay | grep -v "/mnt/docker-data"

Expected output: nothing. Any output indicates containerd still writing to root disk.

Check disk usage on root filesystem:

df -h /

Purpose: Verify root disk usage is stable and not increasing.

Check disk usage on LVM volume:

df -h /mnt/docker-data

Purpose: Confirm LVM volume is being used for container storage and showing increased usage after pulling images.

### 6. Test Functionality

Pull a test image to verify storage works:

docker pull alpine:latest

Purpose: Test that Docker can pull images using the new storage location. Monitor df -h during this operation to ensure root disk does not increase.

Run a test container:

docker run --rm alpine echo "Storage migration successful"

Purpose: Verify containers can start and run correctly with the new storage configuration.

### 7. Clean Up

After confirming everything works correctly for 24-48 hours, remove the backup:

sudo rm -rf /var/lib/containerd.bak

Purpose: Free up space on root disk by removing the old containerd data backup.

Verify root disk space recovered:

df -h /

Purpose: Confirm that removing the backup increased available space on root disk.

### 8. Monitoring Commands

Watch root disk usage in real-time:

watch -n1 df -h

Check all active overlay mounts with details:

cat /proc/mounts | grep overlay

Purpose: View complete overlay mount information including lowerdir, upperdir, and workdir paths. This is the most reliable way to confirm where overlay data resides.


## Key Lessons

1. Docker's data-root setting only affects /var/lib/docker. Containerd data at /var/lib/containerd must be moved separately.

2. Overlay mounts reveal the true storage location. Always check mount | grep overlay after configuration changes.

3. Both Docker and containerd must be stopped before moving data to prevent file corruption.

4. The LVM volume must be at least as large as the combined data from /var/lib/docker and /var/lib/containerd. In this case, both disks were 20GB which is sufficient but does not allow for much growth.

5. Regular monitoring with df -h and mount | grep overlay helps detect storage issues early.

## Prevention

To avoid this issue in the future:

- Always configure both Docker and containerd storage paths when provisioning a new server
- Use LVM for all container storage to allow future expansion
- Monitor disk usage with automated alerts when usage exceeds 80 percent
- Run docker system prune -a periodically to remove unused images and layers
- Store containerd snapshots on separate volume from Docker metadata for better performance and management