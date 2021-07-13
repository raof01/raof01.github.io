---
layout: post
title:  "使用Wireguard搭建VPN"
date:   2021-07-13 13:29:05 -0700
categories: server
---
## Installation on Debian 10
### Installation
 1. Update & Upgrade system
```bash
sudo apt update
sudo apt upgrade -y
```
 2. Enable backports repo
```bash
sudo sh -c "echo 'deb http://deb.debian.org/debian buster-backports main contrib non-free' > /etc/apt/sources.list.d/buster-backports.list"
```
 3. Update repo
```bash
sudo apt update
```
 4. Install wireguard on client & server
```bash
sudo apt install wireguard -y
```

### Key Generation
```bash
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

## Server Configuration
### Wireguard Server Configuration
 1. Create `/etc/wireguard/wg0.conf` with contents:
```
[Interface]
Address = 10.0.0.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
```
 2. Replace `SERVER_PRIVATE_KEY` with result of command
```bash
sudo cat /etc/wireguard/privatekey
```
 3. Replace `ens3` with result of command:
```bash
ip -o -4 route show to default | awk '{print $5}'
```
 4. Make the `privatekey` and `wg0.conf` only readable to user:
```bash
sudo chmod 600 /etc/wireguard/{privatekey,wg0.conf}
```
 5. Start the interface:
```bash
sudo wg-quick up wg0
```
 6. Enable service:
```bash
sudo systemctl enable wg-quick@wg0
```

### Server Networking and Firewall Configuration
 1. Uncommend or add line `net.ipv4.ip_forward=1` to `/etc/sysctl.conf`, then apply the change:
```bash
sudo sysctl -p
```
 2. UFW allow:
```bash
sudo ufw allow 51820/udp
```

## Client Configuration
### Wireguard Configuration
 1. Create `/etc/wireguard/wg0.conf` with contents:
```bash
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/32
[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_IP_ADDRESS:51820
AllowedIPs = 0.0.0.0/0
```
 2. Replace `CLIENT_PRIVATE_KEY` with result of command
```bash
sudo cat /etc/wireguard/privatekey
```

 3. Replace `SERVER_PUBLIC_KEY` with result of running command on server
```bash
sudo cat /etc/wireguard/publickey
```
 4. Replace `SERVER_IP_ADDRESS` with public IP address of server

## Add the Client Peer to the Server
On server:
```bash
sudo wg set wg0 peer CLIENT_PUBLIC_KEY allowed-ips 10.0.0.2
```
Replace `CLIENT_PUBLIC_KEY` with result of running command on client
```bash
sudo cat /etc/wireguard/publickey
```
Then run the command on client:
```bash
sudo wg-quick up wg0
```
If this command failed with:
```
[#] ip link add wg0 type wireguard
RTNETLINK answers: Operation not supported
Unable to access interface: Protocol not supported
[#] ip link delete dev wg0
Cannot find device "wg0"
```
then
```bash
sudo apt install wireguard-dkms wireguard-tools linux-headers-$(uname -r) -y
sudo wg-quick up wg0
```
Finally enable service:
```bash
sudo systemctl enable wg-quick@wg0
```
