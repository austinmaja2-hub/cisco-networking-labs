# Lab 04 — DHCP & Network Services

## Overview

This lab demonstrates how Dynamic Host Configuration Protocol (DHCP) can automatically provide network configuration to client devices.

Cisco Packet Tracer was used to configure a Cisco 1941 router as a DHCP server, connect four PCs through a Cisco 2960 switch, and verify automatic IP assignment and network connectivity.

## Objectives

* Understand DHCP
* Configure a router as a DHCP server
* Create a DHCP pool
* Exclude reserved IP addresses
* Automatically assign IP addresses to clients
* Automatically provide default gateway information
* Automatically provide DNS server information
* Verify DHCP leases
* Test network connectivity
* Renew a DHCP lease
* Understand DHCP's relevance to cybersecurity

## Network Topology

The lab contains:

* 1 × Cisco 1941 Router
* 1 × Cisco 2960 Switch
* 4 × PCs

Topology:

```text
                    Router0
                 192.168.40.1
                DHCP Server
                     │
                     │
                  Switch0
            ┌────────┼────────----------
            │        │        │        |
           PC0      PC1      PC2      PC3
           .2       .3       .4       .5
```

## Network Addressing

| Device  | IP Address   | Subnet Mask   | Default Gateway | Assignment |
| ------- | ------------ | ------------- | --------------- | ---------- |
| Router0 | 192.168.40.1 | 255.255.255.0 | —               | Static     |
| PC0     | 192.168.40.2 | 255.255.255.0 | 192.168.40.1    | DHCP       |
| PC1     | 192.168.40.3 | 255.255.255.0 | 192.168.40.1    | DHCP       |
| PC2     | 192.168.40.4 | 255.255.255.0 | 192.168.40.1    | DHCP       |
| PC3     | 192.168.40.5 | 255.255.255.0 | 192.168.40.1    | DHCP       |

Network:

```text
192.168.40.0/24
```

## Router Interface Configuration

Router0 was configured with the following LAN interface:

```text
interface gigabitEthernet 0/0
ip address 192.168.40.1 255.255.255.0
no shutdown
```

The interface was verified as:

```text
GigabitEthernet0/0    192.168.40.1    up    up
```

## DHCP Configuration

The router was configured as the DHCP server.

First, the router's own IP address was excluded from the DHCP pool:

```text
ip dhcp excluded-address 192.168.40.1
```

A DHCP pool was then created:

```text
ip dhcp pool LAB4-LAN
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 8.8.8.8
exit
```

### Configuration Explanation

**Excluded address**

```text
ip dhcp excluded-address 192.168.40.1
```

Prevents the router's own address from being assigned to another device.

**DHCP pool**

```text
ip dhcp pool LAB4-LAN
```

Creates a DHCP pool named `LAB4-LAN`.

**Network**

```text
network 192.168.40.0 255.255.255.0
```

Defines the network from which DHCP addresses are allocated.

**Default gateway**

```text
default-router 192.168.40.1
```

Tells clients to use Router0 as their default gateway.

**DNS server**

```text
dns-server 8.8.8.8
```

Provides clients with Google's public DNS resolver.

## DHCP Address Assignment

After selecting DHCP on each PC, the router automatically assigned:

```text
PC0 → 192.168.40.2
PC1 → 192.168.40.3
PC2 → 192.168.40.4
PC3 → 192.168.40.5
```

The clients also received:

```text
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.40.1
DNS Server:       8.8.8.8
```

## DHCP Verification

The following command was used on Router0:

```text
show ip dhcp binding
```

The output confirmed four DHCP bindings corresponding to the four PCs.

A DHCP binding associates a leased IP address with a specific client.

## Client Verification

On PC0:

```text
ipconfig /all
```

was used to verify that DHCP had provided the required network configuration.

The following information was confirmed:

* IPv4 address
* Subnet mask
* Default gateway
* DNS server
* DHCP server

## Connectivity Testing

### PC0 → PC1

```text
ping 192.168.40.3
```

Result:

```text
4 sent, 4 received
```

### PC0 → Default Gateway

```text
ping 192.168.40.1
```

Result:

```text
4 sent, 4 received
```

These tests confirmed that the DHCP-assigned addressing allowed communication between the PCs and the router.

## DHCP Renewal

The following command was executed on PC0:

```text
ipconfig /renew
```

PC0 retained:

```text
192.168.40.2
```

This demonstrated that a DHCP lease can be renewed without necessarily changing the client's IP address.

## Important Terminology

### DHCP

**Dynamic Host Configuration Protocol**

A protocol that automatically provides network configuration to clients.

### DHCP Server

A device that manages and distributes network configuration to DHCP clients.

In this lab, **Router0** acted as the DHCP server.

### DHCP Client

A device that requests network configuration from a DHCP server.

The four PCs were DHCP clients.

### DHCP Pool

A range of addresses that a DHCP server can allocate to clients.

### DHCP Lease

A temporary assignment of an IP address to a DHCP client.

### DHCP Binding

The DHCP server's record linking an assigned IP address to a particular client.

### Static IP Address

An IP address manually configured on a device.

### Dynamic IP Address

An IP address automatically assigned through DHCP.

### Default Gateway

The device a host sends traffic to when the destination is outside its local network.

### DNS

**Domain Name System**

A system that translates domain names into IP addresses.

### DORA

The four basic stages of DHCP address assignment:

```text
Discover
Offer
Request
Acknowledgment
```

## Cybersecurity Relevance

DHCP is an important network service that cybersecurity professionals need to understand because it affects how devices receive and use network configuration.

During authorized network assessments, understanding DHCP can help analysts understand how hosts are dynamically assigned addresses and how clients obtain gateway and DNS information.

DHCP also has security considerations.

Examples include:

* DHCP starvation
* Rogue DHCP servers
* Unauthorized network devices
* Misconfigured DHCP scopes
* Improper network segmentation

These concepts can be explored later in controlled cybersecurity labs.

## Verification Commands

### Check router interfaces

```text
show ip interface brief
```

### Check DHCP bindings

```text
show ip dhcp binding
```

### Display DHCP configuration

```text
show running-config | section dhcp
```

### Check client configuration

```text
ipconfig /all
```

### Renew DHCP lease

```text
ipconfig /renew
```

### Test connectivity

```text
ping <destination-ip>
```

## What I Learned

Through this lab I learned how to:

* Configure a Cisco router as a DHCP server
* Create a DHCP pool
* Exclude reserved IP addresses
* Automatically assign IPv4 addresses
* Automatically provide subnet masks
* Automatically provide default gateways
* Automatically provide DNS server information
* Verify DHCP bindings
* Understand DHCP leases
* Renew a DHCP lease
* Test connectivity using ICMP/ping
* Understand the cybersecurity implications of DHCP

## Files

* `dhcp-network-services.pkt` — Cisco Packet Tracer project
* `topology.png` — Network topology

## Status

**Completed**

Lab 04 successfully demonstrates DHCP configuration, automatic IP assignment, DHCP lease management, and network connectivity using Cisco Packet Tracer.
