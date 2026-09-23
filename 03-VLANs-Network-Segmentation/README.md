# Lab 03 — VLANs & Network Segmentation

## Overview

This lab demonstrates how VLANs can be used to logically segment a network and how inter-VLAN communication can be enabled using router-on-a-stick.

The lab was built using Cisco Packet Tracer.

## Objectives

* Understand VLANs and network segmentation
* Create and configure multiple VLANs
* Assign switch ports to VLANs
* Configure access ports
* Configure a trunk link
* Configure 802.1Q encapsulation
* Configure router subinterfaces
* Configure default gateways
* Implement inter-VLAN routing
* Verify connectivity and routing
* Understand the cybersecurity relevance of network segmentation

## Network Topology

The network contains:

* 1 × Cisco 2960 switch
* 1 × Cisco 1941 router
* 6 × PCs

The PCs are divided into three departments:

* TECH
* HR
* ADMIN

## VLAN Design

| VLAN ID | Name  | Network         | Hosts    |
| ------- | ----- | --------------- | -------- |
| 10      | TECH  | 192.168.10.0/24 | PC0, PC1 |
| 20      | HR    | 192.168.20.0/24 | PC2, PC3 |
| 30      | ADMIN | 192.168.30.0/24 | PC4, PC5 |

## IP Addressing

| Device | VLAN | IP Address    | Subnet Mask   | Default Gateway |
| ------ | ---: | ------------- | ------------- | --------------- |
| PC0    |   10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1    |
| PC1    |   10 | 192.168.10.20 | 255.255.255.0 | 192.168.10.1    |
| PC2    |   20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1    |
| PC3    |   20 | 192.168.20.20 | 255.255.255.0 | 192.168.20.1    |
| PC4    |   30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1    |
| PC5    |   30 | 192.168.30.20 | 255.255.255.0 | 192.168.30.1    |

## Switch Port Assignments

| Switch Port | Device  |       VLAN | Port Type |
| ----------- | ------- | ---------: | --------- |
| Fa0/1       | PC0     |         10 | Access    |
| Fa0/2       | PC1     |         10 | Access    |
| Fa0/3       | PC2     |         20 | Access    |
| Fa0/4       | PC3     |         20 | Access    |
| Fa0/5       | PC4     |         30 | Access    |
| Fa0/6       | PC5     |         30 | Access    |
| Fa0/24      | Router0 | 10, 20, 30 | Trunk     |

## VLAN Configuration

The following VLANs were created on the switch:

```text
vlan 10
name TECH

vlan 20
name HR

vlan 30
name ADMIN
```

The PC-facing interfaces were configured as access ports and assigned to their respective VLANs.

## Trunk Configuration

FastEthernet0/24 was configured as the trunk between the switch and router:

```text
interface fastEthernet 0/24
switchport mode trunk
```

The trunk allows multiple VLANs to travel between the switch and router.

## Router-on-a-Stick Configuration

The router uses one physical interface with multiple logical subinterfaces.

```text
interface gigabitEthernet 0/0
no shutdown

interface gigabitEthernet 0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet 0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface gigabitEthernet 0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```

Each subinterface acts as the default gateway for its VLAN.

## Connectivity Testing

### Same-VLAN Testing

TECH:

```text
PC0 → PC1
4 sent, 4 received
```

HR:

```text
PC2 → PC3
4 sent, 4 received
```

ADMIN:

```text
PC4 → PC5
4 sent, 4 received
```

### Inter-VLAN Testing

TECH → HR:

```text
PC0 → 192.168.20.10
4 sent, 4 received
```

TECH → ADMIN:

```text
PC0 → 192.168.30.10
4 sent, 4 received
```

HR → ADMIN:

```text
PC2 → 192.168.30.10
4 sent, 4 received
```

These tests confirmed that the router successfully routed traffic between the three VLANs.

## Verification Commands

### Display VLAN configuration

```text
show vlan brief
```

### Display trunk status

```text
show interfaces trunk
```

### Display router interfaces

```text
show ip interface brief
```

### Display routing table

```text
show ip route
```

The routing table showed directly connected routes for:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
```

## Key Terminology

### VLAN

A Virtual Local Area Network is a logical network that separates devices at Layer 2 even when they are connected to the same physical switch.

### Access Port

A switch port assigned to a single VLAN, normally used for end devices such as PCs.

### Trunk Port

A port capable of carrying traffic belonging to multiple VLANs.

### 802.1Q

An IEEE standard used to add VLAN tags to Ethernet frames.

### Subinterface

A logical interface created within a physical router interface.

### Inter-VLAN Routing

The process of routing traffic between different VLAN networks.

### Router-on-a-Stick

A network design where one physical router interface uses multiple subinterfaces to route between VLANs.

### Default Gateway

The IP address a host uses when it needs to communicate with a device outside its local network.

### ARP

Address Resolution Protocol. It is used to discover the MAC address associated with an IPv4 address on a local network.

## Cybersecurity Relevance

Network segmentation is an important cybersecurity concept.

Separating TECH, HR and ADMIN into different VLANs reduces the size of each Layer 2 broadcast domain and provides a foundation for controlling communication between departments.

However, VLAN segmentation alone does not prevent all communication between networks.

Because inter-VLAN routing is enabled, traffic can pass through the router. In a production environment, additional security controls such as Access Control Lists (ACLs), firewalls and monitoring could be used to restrict unnecessary communication.

For example, security policies could restrict which systems in the TECH network are allowed to access sensitive ADMIN or HR resources.

## What I Learned

Through this lab I learned how to:

* Create VLANs
* Assign switch ports to VLANs
* Configure access ports
* Configure trunk ports
* Understand 802.1Q VLAN tagging
* Configure router subinterfaces
* Configure default gateways
* Implement router-on-a-stick
* Perform inter-VLAN routing
* Verify VLAN and routing configuration
* Use ping to test network connectivity
* Understand the relationship between VLANs, routing and network security

## Files

* `vlan-network-segmentation.pkt` — Cisco Packet Tracer project
* `topology.png` — Network topology

## Status

**Completed**

Lab 03 successfully demonstrates VLAN segmentation and inter-VLAN routing using Cisco Packet Tracer.
