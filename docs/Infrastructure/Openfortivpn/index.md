# OpenFortiVPN Installation, Configuration, and NAT Setup on Ubuntu

This document explains how to install, configure, and run
**OpenFortiVPN** on Ubuntu, including setting up **NAT routing through
`ppp0`** and configuring firewall rules using **UFW**.

## 1. Install Dependencies

``` bash
sudo apt update -y
sudo apt install -y \
  autoconf \
  automake \
  build-essential \
  pkg-config \
  libssl-dev \
  libtool \
  git \
  ppp
```

## 2. Clone and Build OpenFortiVPN

``` bash
git clone https://github.com/adrienverge/openfortivpn.git
cd openfortivpn
git checkout v1.23.1   # or latest tag
./autogen.sh
./configure
make
sudo make install
```

Verify installation:

``` bash
openfortivpn --version
```

## 3. Configure OpenFortiVPN

``` bash
mkdir -p /etc/openfortivpn
touch /etc/openfortivpn/config
chmod 600 /etc/openfortivpn/config
```

Edit `/etc/openfortivpn/config`:

    host = 192.168.1.1
    port = 123
    username = username
    password = mypassword
    trusted-cert = EXAMPLE_CERT_VALUE

## 4. Test VPN Interface

``` bash
openfortivpn -c /etc/openfortivpn/config --persistent=10
ip a show ppp0
pkill pppd
```

## 5. Setup Systemd Service

Create: `/etc/systemd/system/openfortivpn-nat.service`

``` ini
[Unit]
Description=OpenFortiVPN with NAT
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/openfortivpn -c /etc/openfortivpn/config --persistent=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable:

``` bash
systemctl daemon-reload
systemctl enable openfortivpn-nat
systemctl start openfortivpn-nat
```

## 6. Enable IP Forwarding

Edit:

    /etc/ufw/sysctl.conf
    net/ipv4/ip_forward=1

Apply:

``` bash
sysctl -p /etc/ufw/sysctl.conf
```

## 7. Configure UFW & IPTables

``` bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

iptables -A INPUT -p tcp --dport 2112 -j ACCEPT
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -p tcp --dport 10050 -j ACCEPT

iptables -A FORWARD -d 192.168.112.50/32 -i ens33 -o ppp0 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 192.168.112.50/32 -i ppp0 -o ens33 -m conntrack --ctstate ESTABLISHED -j ACCEPT

iptables -A OUTPUT -o lo -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
iptables -A OUTPUT -d 109.125.133.139/32 -p tcp --dport 11443 -j ACCEPT
```

## 8. Verify Configuration

``` bash
ufw status verbose
iptables -t nat -L -v -n
ip route
```

## 9. Optional Static Route

``` bash
sudo ip route add 192.168.242.0/24 via 192.168.99.254 dev ens33
```

Netplan:

``` yaml
network:
  version: 2
  ethernets:
    ens33:
      addresses:
        - 192.168.99.95/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.99.254
        - to: 192.168.242.0/24
          via: 192.168.99.254
          table: 0
      nameservers:
        addresses:
          - 192.168.90.10
```

Apply:

``` bash
netplan apply
```
