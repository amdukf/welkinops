# Docker and Containerd Storage Migration to LVM

## 1. Problem Statement

The root disk (20GB) was filling up because Docker images, containers, and volumes were stored on the root partition. The server had an external 20GB disk (vdb) configured as an LVM logical volume. The goal was to move all Docker and containerd data to the LVM volume to prevent the root disk from filling up.

## 2. Understanding Docker and Containerd Storage

Docker uses two separate storage locations:

| Location | Purpose |
|----------|---------|
| `/var/lib/docker` | Docker metadata, container configurations, volumes, and network settings |
| `/var/lib/containerd` | Container image layers, snapshots, and overlay filesystem data |

Important: Changing only Docker's `data-root` setting does NOT move containerd data. Containerd continues writing image layers to `/var/lib/containerd` on the root disk, causing disk usage to increase even after Docker is reconfigured.

## 3. System Configuration

| Component | Device | Mount Point | Size |
|-----------|--------|-------------|------|
| Root disk | `/dev/vda1` | `/` | 20GB |
| External disk | `/dev/vdb` | N/A | 20GB |
| LVM Volume | `/dev/mapper/yellowstone-lv_docker` | `/mnt/docker-data` | 20GB |

## 4. LVM Setup

Install LVM tools if not already installed:

```bash
apt update
apt install -y lvm2
```

Create physical volume and volume group:

```bash
pvcreate /dev/vdb
vgcreate yellowstone /dev/vdb
```

Create logical volume using all available space:

```bash
lvcreate -l 100%FREE -n lv_docker yellowstone
```

Format the logical volume:

```bash
mkfs.ext4 /dev/yellowstone/lv_docker
```

Create mount point and mount:

```bash
mkdir -p /mnt/docker-data
mount /dev/yellowstone/lv_docker /mnt/docker-data
```

Make mount permanent by adding to `/etc/fstab`:

```bash
# Get the UUID
blkid /dev/yellowstone/lv_docker

# Add to /etc/fstab (replace UUID with actual value)
echo "UUID=your-uuid-here /mnt/docker-data ext4 defaults,nofail 0 2" >> /etc/fstab
```

Verify LVM setup:

```bash
lsblk
df -h /mnt/docker-data
```

## 5. Migration Procedure

### 5.1 Pre-Migration Checks

Check current disk usage:

```bash
df -h
```

Check current Docker storage location:

```bash
docker info | grep "Docker Root Dir"
```

Check block device layout:

```bash
lsblk
```

### 5.2 Stop All Services

Stop Docker and containerd to prevent writes during migration:

```bash
sudo systemctl stop docker docker.socket containerd
```

Verify services are stopped:

```bash
sudo systemctl status docker containerd
```

### 5.3 Configure Docker to Use LVM

Create Docker daemon configuration:

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "data-root": "/mnt/docker-data"
}
EOF
```

### 5.4 Move Docker Data to LVM

Copy Docker data to LVM volume:

```bash
sudo rsync -avxP /var/lib/docker/ /mnt/docker-data/
```

Rename original directory as backup:

```bash
sudo mv /var/lib/docker /var/lib/docker.bak
```

### 5.5 Move Containerd Data to LVM

Create containerd directory on LVM:

```bash
sudo mkdir -p /mnt/docker-data/containerd
```

Copy containerd data to LVM:

```bash
sudo rsync -avxP /var/lib/containerd/ /mnt/docker-data/containerd/
```

Rename original directory as backup:

```bash
sudo mv /var/lib/containerd /var/lib/containerd.bak
```

### 5.6 Reconfigure Containerd

Generate default containerd configuration:

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Edit the configuration to use LVM path:

```bash
sudo sed -i 's|root = ".*"|root = "/mnt/docker-data/containerd"|' /etc/containerd/config.toml
```

Verify the change:

```bash
grep "^root" /etc/containerd/config.toml
```

Expected output: `root = "/mnt/docker-data/containerd"`

### 5.7 Restart Services

Start containerd first:

```bash
sudo systemctl start containerd
```

Then start Docker:

```bash
sudo systemctl start docker
```

Check service status:

```bash
sudo systemctl status containerd docker
```

### 5.8 Verify Migration

Verify Docker storage location:

```bash
docker info | grep "Docker Root Dir"
```

Expected output: `Docker Root Dir: /mnt/docker-data`

Verify containerd overlays are using LVM path:

```bash
mount | grep overlay | head -5
```

All `lowerdir` and `upperdir` paths should show `/mnt/docker-data/containerd/...` not `/var/lib/containerd/...`

Verify no containerd paths remain on root disk:

```bash
mount | grep overlay | grep -v "/mnt/docker-data"
```

Expected output: nothing.

Check root disk usage (should be stable):

```bash
df -h /
```

Check LVM volume usage (should increase as images are pulled):

```bash
df -h /mnt/docker-data
```

### 5.9 Test Functionality

Pull a test image:

```bash
docker pull alpine:latest
```

Run a test container:

```bash
docker run --rm alpine echo "Storage migration successful"
```

List images to confirm they are accessible:

```bash
docker images
```

## 6. Cleanup

After confirming everything works correctly for 24-48 hours, remove the backups to free root disk space:

```bash
sudo rm -rf /var/lib/docker.bak
sudo rm -rf /var/lib/containerd.bak
```

Verify root disk space recovered:

```bash
df -h /
```

## 7. Monitoring Commands

Monitor root disk usage in real-time:

```bash
watch -n1 df -h /
```

Monitor LVM disk usage in real-time:

```bash
watch -n1 df -h /mnt/docker-data
```

Check all active overlay mounts with details:

```bash
cat /proc/mounts | grep overlay
```

Check Docker storage breakdown:

```bash
docker system df -v
```

## 8. Troubleshooting

### Rollback to previous configuration

If migration fails, rollback using backups:

```bash
sudo systemctl stop docker containerd
sudo rm -rf /var/lib/docker /var/lib/containerd
sudo mv /var/lib/docker.bak /var/lib/docker
sudo mv /var/lib/containerd.bak /var/lib/containerd
sudo systemctl start containerd docker
```

## 9. Key Lessons

1. Docker's `data-root` setting only affects `/var/lib/docker`. Containerd data at `/var/lib/containerd` must be moved separately.

2. Overlay mounts reveal the true storage location. Always check `mount | grep overlay` after configuration changes.

3. Both Docker and containerd must be stopped before moving data to prevent file corruption.

4. The LVM volume must be at least as large as the combined data from `/var/lib/docker` and `/var/lib/containerd`.

5. Regular monitoring with `df -h` and `mount | grep overlay` helps detect storage issues early.

6. The `rsync` command with `-avxP` flags preserves permissions, copies recursively, stays within filesystem boundaries, and shows progress.

## 10. Prevention

To avoid this issue in the future:

- Always configure both Docker and containerd storage paths when provisioning a new server
- Use LVM for all container storage to allow future expansion
- Monitor disk usage with automated alerts when usage exceeds 80 percent
- Run `docker system prune -a` periodically to remove unused images and layers
- Store containerd snapshots on a separate volume from Docker metadata for better performance and management
- Include both `/var/lib/docker` and `/var/lib/containerd` in backup strategies

## Appendix A: Quick Reference Commands

| Operation | Command |
|-----------|---------|
| Check disk usage | `df -h` |
| Check block devices | `lsblk` |
| Check LVM volumes | `lvs` |
| Check LVM volume groups | `vgs` |
| Check overlay mounts | `mount \| grep overlay` |
| Check Docker storage root | `docker info \| grep "Root Dir"` |
| Check Docker disk usage | `docker system df` |
| Clean unused Docker data | `docker system prune -a` |
