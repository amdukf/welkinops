# VM Preparation in UTM

This guide walks through preparing an Ubuntu Server 24.04 virtual machine in [UTM](https://mac.getutm.app) that is compatible with [Vagrant](https://www.vagrantup.com) using the [`vagrant-utm`](https://github.com/naveenrajm7/vagrant_utm) provider.

---

## 1. Create a New VM in UTM

- **Architecture:** `ARM64 (aarch64)` *(if using Apple Silicon)*
- **System:** Ubuntu Server 24.04 ISO
- **VM Type:** Virtualized (not emulated)

### Configuration

| Setting                  | Value                             |
|--------------------------|-----------------------------------|
| Memory (RAM)             | ≥ 1024 MB                         |
| CPUs                     | ≥ 1                               |
| Shared Directory         | Optional                          |
| Display                  | VGA or Virtio (leave default)     |
| Sound, USB, Clipboard    | Not needed                        |

---

## 2. Network Interface Setup

**This is critical for Vagrant compatibility:**

| Adapter | Type           | Purpose                |
|---------|----------------|------------------------|
| 1       | Shared Network | NAT-like for internet  |
| 2       | Emulated VLAN  | For port forwarding (Vagrant uses this) |

You can configure this under **UTM > Edit VM > Network**.

---

## 3. Install Ubuntu Server Normally

- Proceed through the normal Ubuntu Server 24.04 installer.
- Choose minimal or standard installation.
- Set a temporary username and password (e.g., `vagrant` / `vagrant`).

---

## 4. Add the `vagrant` User and SSH Keys

After installation, log into the VM and run:

```bash
sudo apt update
sudo apt install -y curl sudo openssh-server
sudo useradd -m -s /bin/bash vagrant # (if you didn't create user in section 3 )
echo "vagrant ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/vagrant
sudo mkdir -p /home/vagrant/.ssh
curl -fsSL https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub \
  -o /home/vagrant/.ssh/authorized_keys
sudo chown -R vagrant:vagrant /home/vagrant/.ssh
sudo chmod 700 /home/vagrant/.ssh
sudo chmod 600 /home/vagrant/.ssh/authorized_keys
```

## 5. Enable and Start SSH

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
ss -tlnp | grep :22
```

## 6. Install and enable QEMU Guest Agent inside your Ubuntu VM

```bash
sudo apt update
sudo apt install -y qemu-guest-agent
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
```
