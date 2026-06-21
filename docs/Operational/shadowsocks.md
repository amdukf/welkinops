# Shadowsocks connection from another server

Server A: your shadowsocks-libev server

Server B: another server

Goal: on Server B, route traffic through Server A’s SOCKS5 proxy provided by Shadowsocks

```
sudo apt update
sudo apt install shadowsocks-libev -y
```

### Configure client on Server B

```
sudo vim /etc/shadowsocks-libev/client.json
```

```
{
    "server": "SERVER_A_IP",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "YOUR_PASSWORD",
    "method": "aes-256-gcm"
}
```

Start SOCKS proxy on Server B
```
ss-local -c /etc/shadowsocks-libev/client.json
```


### Use the proxy
```
curl --socks5 127.0.0.1:1080 https://ifconfig.me
```