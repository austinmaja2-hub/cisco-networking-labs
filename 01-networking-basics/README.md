# Lab 01 — Basic LAN

## Objective

Build a basic Local Area Network (LAN) using two PCs and a Cisco 2960 switch, configure IPv4 addresses, and verify connectivity between the devices.

## Topology

```text
PC0 ───────── Switch ───────── PC1
```

## Devices

* 2 × PCs
* 1 × Cisco 2960 Switch
* Copper Straight-Through Ethernet cables

## IP Addressing

| Device | IPv4 Address | Subnet Mask   |
| ------ | ------------ | ------------- |
| PC0    | 192.168.1.10 | 255.255.255.0 |
| PC1    | 192.168.1.20 | 255.255.255.0 |

Both devices are on the same network:

`192.168.1.0/24`

## Configuration

PC0 and PC1 were configured with static IPv4 addresses.

No default gateway was required because both devices are communicating within the same local network.

## Connectivity Test

From PC0:

```text
ping 192.168.1.20
```

The ping was successful, confirming connectivity between PC0 and PC1.

## Switch Verification

The switch MAC address table was inspected using:

```text
enable
show mac address-table
```

The switch learned the MAC addresses of the connected PCs and associated them with their respective switch ports.

## Terminology Learned

### LAN

A Local Area Network connecting devices within a limited geographical area.

### IP Address

A logical address used to identify a device on an IP network.

### MAC Address

A Layer 2 address associated with a network interface and used for communication on the local network.

### Switch

A network device that connects devices within a LAN and forwards Ethernet frames based on MAC addresses.

### Subnet Mask

Defines the network and host portions of an IPv4 address.

### Default Gateway

A device, normally a router, used to reach destinations outside the local network.

### Ping

A network connectivity test that uses ICMP Echo Request and Echo Reply messages.

### Ethernet

A family of technologies used for wired local-area networking.

### Frame

A Layer 2 data unit used by Ethernet networks.

### Packet

A Layer 3 data unit used by IP networks.

## What I Learned

* How to build a basic LAN in Cisco Packet Tracer.
* How to connect PCs to a switch using Ethernet.
* How to configure static IPv4 addresses.
* How subnet masks determine network membership.
* How to test connectivity using `ping`.
* How switches learn MAC addresses.
* How MAC addresses are associated with switch ports.
* The basic relationship between Layer 1, Layer 2, and Layer 3 networking.

## Commands Used

```text
ping 192.168.1.20
ipconfig /all
enable
show mac address-table
```

## Lab Status

**Completed *
