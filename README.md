# sirazazel's Homelab

The main goal of this project is experimenting and learning about new technologies. Although, all services here hosted need to have a function and provide a direct benefit to the user.

## Overview, status and long-term goals

### Status

### Network

- pfSense Firewall
- 4x Ubiquiti UniFi UAP-AC v2
- 2x TP-Link TL-SG3210

### Compute

- 1x HP ProDesk 800 G3 USFF
    - CPU: Intel Pentium G4400T @ 2.90GHz
    - RAM: 4Gb
    - SSD: 128Gb
    - Network: 2x Intel i210

- 1x Dell Optiplex 790 Gen 4
    - CPU: Intel Core i3 2100 @ 3.10GHz
    - RAM: 16Gb
    - SSD: 120Gb
    - Network: 1x 1Gbps NIC

- 1x Synology DS119j
    - HDD: 1x 2Tb

### Features

- [ ] Fast networking on all points of the house.
- [X] VPN without port forwarding.
- [X] Automatic DNS configuration.
- [ ] Single sign-on.
- [ ] Active Directory domain.
- [ ] Distributed storage.
- [ ] Automatic backups.
- [ ] 

### Tech Stack

|                                                                           |Name           |Description                        |
|---------------------------------------------------------------------------|---------------|-----------------------------------|
|<img src="assets\logos\pfSenselogo.png" alt="drawing" width="100"/>        |pfSense        |Router and firewall                |
|<img src="assets\logos\xcplogo.png" alt="drawing" width="30"/>             |XCP-NG         |Hypervisor                         |
|<img src="assets\logos\xologo.png" alt="drawing" width="30"/>              |Xen Orchestra  |                                   |
|<img src="assets\logos\pfBlockerlogo.png" alt="drawing" width="30"/>       |pfBlockerNG    |DNS Sinkhole                       |
|<img src="assets\logos\cloudflarelogo.png" alt="drawing" width="90"/>      |Cloudflare     |Domain registrar, DNS              |
|<img src="assets\logos\fedora-coreos-logo.png" alt="drawing" width="80"/>  |Fedora CoreOS  |Operating system for nodes         |
|                                                                           |Fedora Server  |                                   | 
|<img src="assets\logos\caddylogo.png" alt="drawing" width="100"/>          |Caddy          |Reverse proxy                      |
|<img src="assets\logos\letsencrypt.png" alt="drawing" width="100"/>        |Let's Encrypt  |                                   |