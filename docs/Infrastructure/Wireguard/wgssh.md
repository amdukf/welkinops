# Securing SSH Access via WireGuard VPN

## 1. Install WireGuard
Update package list and install WireGuard.

    apt update
    apt install wireguard -y

## 2. Generate Keys for Server and Client
Generate a private/public key pair. The private key is used in the `[Interface]` section, the public key is shared with the peer.

    wg genkey | tee privatekey | wg pubkey > publickey

Retrieve the keys:

    cat privatekey
    cat publickey

## 3. Configure the WireGuard Interface
Create the file `/etc/wireguard/wg0.conf` with the following content:
```
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.10.0.1/24
ListenPort = 51920

PostUp = iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.10.0.2/32
```

> Replace `SERVER_PRIVATE_KEY` with the server’s private key, `CLIENT_PUBLIC_KEY` with the client’s public key, and `ens3` with the name of your public network interface.

## 4. Enable IP Forwarding
Add `net.ipv4.ip_forward=1` to `/etc/sysctl.conf` and apply the changes.

    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
    sysctl -p

## 5. Start and Enable the WireGuard Interface
Bring up the interface and enable it to start at boot.

    wg-quick up wg0
    systemctl enable wg-quick@wg0

## 6. Open the WireGuard Port in the Firewall
Allow UDP traffic on the configured port (51920).

    ufw allow 51920/udp

## 7. Verify Connectivity
From the client, ensure the VPN tunnel is operational.

    ping 10.10.0.1

If the ping succeeds, the tunnel is working.

## 8. Restrict SSH to the WireGuard Network

### Option A (simple, recommended)
Remove any blanket deny for port 22, set the default incoming policy to deny, and allow SSH only from the WireGuard subnet.

    sudo ufw delete deny 22
    sudo ufw default deny incoming
    sudo ufw allow from 10.10.0.0/24 to any port 22
    sudo ufw reload

### Option B (explicit deny rule)
If you prefer to keep an explicit `deny 22` rule, insert the allow rule before it.

    sudo ufw insert 1 allow from 10.10.0.0/24 to any port 22

## 9. Troubleshooting Commands
If you need to reset or restart the WireGuard interface, use these commands.

    ip link delete wg0
    wg-quick down wg0
    systemctl restart wg-quick@wg0