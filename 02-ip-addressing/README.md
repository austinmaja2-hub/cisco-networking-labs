# Cisco Networking Lab 02 — IP Addressing and Routing

## Objective

The objective of this lab was to understand IPv4 addressing, subnetting, default gateways, router interfaces, and basic routing between two separate networks using Cisco Packet Tracer.

The lab demonstrates how a router allows devices on different IP networks to communicate.

---

## Topology

```text
                    Router0
                 ┌────────────┐
                 │            │
       G0/0      │            │      G0/1
        │        │            │        │
        │        └────────────┘        │
        │                              │
     Switch0                         Switch1
      /   \                           /   \
     /     \                         /     \
 Laptop0  Laptop1                 Laptop2  Laptop3
```

### Network Structure

```text
Network A                         Network B

192.168.10.0/24                   192.168.20.0/24

Laptop0                            Laptop2
192.168.10.10                      192.168.20.10
     │                                  │
Laptop1                            Laptop3
192.168.10.20                      192.168.20.20
     │                                  │
  Switch0                            Switch1
     │                                  │
     └──── Router0 ────────────────────┘
```

---

## Devices Used

| Device                         | Quantity |
| ------------------------------ | -------: |
| Cisco 1941 Router              |        1 |
| Cisco 2960 Switch              |        2 |
| PCs/Laptops                    |        4 |
| Copper Straight-Through Cables |        6 |

---

## IP Addressing

### Network A

**Network:** `192.168.10.0/24`

| Device      | IP Address      | Subnet Mask     | Default Gateway |
| ----------- | --------------- | --------------- | --------------- |
| Laptop0     | `192.168.10.10` | `255.255.255.0` | `192.168.10.1`  |
| Laptop1     | `192.168.10.20` | `255.255.255.0` | `192.168.10.1`  |
| Router G0/0 | `192.168.10.1`  | `255.255.255.0` | —               |

### Network B

**Network:** `192.168.20.0/24`

| Device      | IP Address      | Subnet Mask     | Default Gateway |
| ----------- | --------------- | --------------- | --------------- |
| Laptop2     | `192.168.20.10` | `255.255.255.0` | `192.168.20.1`  |
| Laptop3     | `192.168.20.20` | `255.255.255.0` | `192.168.20.1`  |
| Router G0/1 | `192.168.20.1`  | `255.255.255.0` | —               |

---

## Router Configuration

The Cisco 1941 router was configured with two GigabitEthernet interfaces.

### G0/0 — Network A

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

### G0/1 — Network B

```text
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
exit
```

---

## Interface Verification

The following command was used:

```text
show ip interface brief
```

The important interfaces showed:

```text
GigabitEthernet0/0    192.168.10.1    up    up
GigabitEthernet0/1    192.168.20.1    up    up
```

### Meaning

**up/up** indicates that both the interface and its line protocol are operational.

The router's VLAN interface was not used in this lab and remained administratively down.

---

## Routing Table Verification

The following command was used:

```text
show ip route
```

The router displayed:

```text
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
L    192.168.10.1/32 is directly connected, GigabitEthernet0/0

C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
L    192.168.20.1/32 is directly connected, GigabitEthernet0/1
```

### Route Types

**C — Connected**

The `C` indicates that the network is directly connected to the router.

For example:

```text
C 192.168.10.0/24 → G0/0
```

means the router can reach the `192.168.10.0/24` network directly through GigabitEthernet0/0.

**L — Local**

The `L` route represents an IP address belonging directly to the router.

For example:

```text
L 192.168.10.1/32
```

represents the router's own G0/0 address.

---

## Connectivity Testing

Connectivity was tested using Cisco Packet Tracer's Simulation Mode.

### Test 1 — Same Network

```text
Laptop2 → Laptop3
```

Result:

**Successful**

Both devices belong to:

```text
192.168.20.0/24
```

Therefore, they can communicate through Switch1 without requiring routing between different networks.

---

### Test 2 — Different Networks

```text
Laptop0 → Laptop2
```

Result:

**Successful**

The packet traveled from:

```text
192.168.10.0/24
```

to:

```text
192.168.20.0/24
```

through Router0.

---

### Test 3 — Different Networks

```text
Laptop3 → Laptop0
```

Result:

**Successful**

This demonstrated that communication was possible in the reverse direction.

---

### Test 4 — Laptop0 → Laptop3

The first simulation attempt failed, but after clearing the previous simulation events and testing again, the packet was successfully delivered.

Final result:

**Successful**

This confirmed connectivity between all four laptops across both networks.

---

## Packet Path

When Laptop0 communicates with Laptop3, the packet travels approximately:

```text
Laptop0
192.168.10.10
     ↓
Switch0
     ↓
Router G0/0
192.168.10.1
     ↓
Router G0/1
192.168.20.1
     ↓
Switch1
     ↓
Laptop3
192.168.20.20
```

The router determines that the destination belongs to the `192.168.20.0/24` network and forwards the packet through G0/1.

---

## Key Terminology

### IPv4 Address

A 32-bit logical address used to identify a device on an IP network.

Example:

```text
192.168.10.10
```

### Subnet Mask

Determines which portion of an IPv4 address identifies the network and which portion identifies the host.

Example:

```text
255.255.255.0
```

This is equivalent to:

```text
/24
```

### Network Address

Identifies the network itself.

Example:

```text
192.168.10.0/24
```

### Default Gateway

The router address that a device uses when communicating with a different network.

Network A:

```text
192.168.10.1
```

Network B:

```text
192.168.20.1
```

### Router

A networking device that connects different IP networks and forwards packets between them.

### Routing

The process of determining where packets should be sent to reach their destination.

### Interface

A network connection on a device.

In this lab:

```text
G0/0
G0/1
```

are router interfaces.

### CIDR

Classless Inter-Domain Routing notation used to represent network prefixes.

Example:

```text
192.168.10.0/24
```

### Packet

A unit of data transmitted across an IP network.

### ICMP

Internet Control Message Protocol. `ping` uses ICMP Echo Request and Echo Reply messages to test connectivity.

---

## Commands Used

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown

interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown

show ip interface brief
show ip route
```

---

## Cybersecurity Relevance

Understanding IP addressing and routing is fundamental to cybersecurity.

Security tools such as Nmap, Wireshark, Scapy, OpenVAS, and Metasploit operate on networks where IP addressing and routing determine how systems communicate.

For example:

```text
nmap 192.168.10.0/24
```

means scanning hosts within the `192.168.10.0/24` network.

Understanding what `/24` represents makes it possible to understand exactly which systems belong to the target network.

Network knowledge is also important when analysing:

* Network traffic
* Firewalls
* Network segmentation
* VLANs
* Routing
* Network reconnaissance
* Vulnerability scanning
* Intrusion detection
* Penetration testing

---

## What I Learned

In this lab, I learned how to:

* Configure IPv4 addresses.
* Understand `/24` subnet notation.
* Identify network addresses.
* Configure router interfaces.
* Enable router interfaces using `no shutdown`.
* Configure default gateways.
* Connect two separate LANs using a router.
* Verify interface status using `show ip interface brief`.
* Read a Cisco routing table using `show ip route`.
* Understand connected (`C`) and local (`L`) routes.
* Test connectivity using ICMP.
* Observe packet movement using Packet Tracer Simulation Mode.
* Understand how routers forward traffic between different subnets.

---

## Lab Status

**Completed successfully **

**Environment:** Cisco Packet Tracer

**Focus:** IPv4 Addressing, Subnetting, Default Gateways and Basic Routing

**Next Topic:** VLANs and Network Segmentation
