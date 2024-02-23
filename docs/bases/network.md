---
layout: default
title: networking
parent: bases
nav_order: 1
---
# Network configuration

## Physical network

### Topology

### Hardware

My current firewall is a modified HP ProDesk Mini from a couple of years ago. For our workload, it really isn’t necessary a lot of computing power, any old desktop or laptop would work, but I have chosen this old tiny PC for three primary reasons:

The first and foremost reason is the low TDP. This machine has to be on 24/7 and I certainly don’t want to give my power company a third of my salary. This machine idles at around 7-10W, so, taking in consideration the median price per kWh of January 2023 **(0,158719€/kWh)**, the cost of mantaining the firewall up and running for a year would be 13,9€.

Secondly, the machine was very cheap on the second hand market, at just 30€. And finally, even if it’s a USFF desktop so there are no expansion ports, in this era of HP desktops they came with a removable port. The default was a VGA connector, but you could change it to a Serial port, another DisplayPort output, but not to an ethernet jack… oficially.

![Firewall modification](/docs/assets/images/firewall/modification.png)

I added a M.2 ethernet adapter to the WLAN port of the machine, and made a custom bracket that fills the void that’s left after the VGA connector is removed. Then, an M.2 SSD was added and the firewall was done.

## 

### VLAN definitions

#### VLAN 1

Management VLAN. This VLAN will host all network infrastructure. Switches, APs, etc.

|Network    |Gateway    |Netmask    |DHCP Range |
|-----------|-----------|-----------|-----------|
|172.16.0.0 |172.16.0.1 |24         |Not enabled|

#### VLAN 3

Multicast network for IPTV.

|Network    |Gateway    |Netmask    |DHCP Range |
|-----------|-----------|-----------|-----------|
|--         |--         |--         |--         |
#### VLAN 10

This VLAN hosts all IoT devices. 

|Network    |Gateway    |Netmask    |DHCP Range |
|-----------|-----------|-----------|-----------|
|192.168.1.0|192.168.1.1|26         |10-60      |

#### VLAN 20

This VLAN hosts trusted devices.

|Network    |Gateway    |Netmask    |DHCP Range |
|-----------|-----------|-----------|-----------|
|10.0.74.0  |10.0.74.1  |24         |200-245    |

### Switchport Configuration

#### SW01
| 1       | 2       | 3      | 4      | 5      | 6      | 7      | 8      | 9      | 10     |
|---------|---------|--------|--------|--------|--------|--------|--------|--------|--------|
| GENERAL | GENERAL | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS |
| 1       | 1       | 20     | 20     | 20     | 20     | 3      | 20     | 20     | 20     |
| Ingress | SW02    | AP 1   |        |        |        | IPTV   |minirex |        |        |

#### SW02
| 1       | 2       | 3      | 4      | 5      | 6      | 7      | 8      | 9      | 10     |
|---------|---------|--------|--------|--------|--------|--------|--------|--------|--------|
| GENERAL | GENERAL | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS | ACCESS |
| 1       | 1       | 20     | 20     | 20     | 20     | 20     | 20     | 20     | 20     |
| Ingress |         | AP 1   |minisarge|       |        | AP 2   | AP 3   |        |        |
