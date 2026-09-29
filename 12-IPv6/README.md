# Lab 12 — IPv6 Networking

## Overview

This lab focuses on implementing and troubleshooting an IPv6 network using Cisco Packet Tracer.

The lab demonstrates IPv6 addressing, IPv6 routing, Neighbor Discovery Protocol (NDP), static IPv6 routes, and end-to-end communication across multiple networks and routers.

The lab also includes troubleshooting a failed IPv6 connection by identifying that IPv6 unicast routing was not enabled on R1.

---

## Objectives

* Configure IPv6 addresses on routers and hosts
* Understand IPv6 `/64` prefixes
* Configure IPv6 interfaces
* Enable IPv6 unicast routing
* Understand IPv6 routing tables
* Understand Neighbor Discovery Protocol (NDP)
* Configure static IPv6 routes
* Configure forwarding and return routes
* Test end-to-end IPv6 connectivity
* Troubleshoot IPv6 connectivity problems

---

## Topology

```text
PC0
 |
 | Fa0/0
 |
Switch0
 |
 | Fa0/2
 |
R1
 | G0/1
 |
 | 2001:DB8:100::/64
 |
 | G0/0
R2
 |
 | G0/1
 |
Switch2
 |
 | Fa0/1
 |
PC1
```

### Device Roles

* **R1** — Router0 in Packet Tracer
* **R2** — Router1 in Packet Tracer
* **PC0** — IPv6 host on LAN A
* **PC1** — IPv6 host on LAN B

---

## IPv6 Addressing Plan

| Device | Interface | IPv6 Address         | Network             |
| ------ | --------- | -------------------- | ------------------- |
| PC0    | Fa0/0     | `2001:DB8:12::10/64` | `2001:DB8:12::/64`  |
| R1     | G0/0      | `2001:DB8:12::1/64`  | `2001:DB8:12::/64`  |
| R1     | G0/1      | `2001:DB8:100::1/64` | `2001:DB8:100::/64` |
| R2     | G0/0      | `2001:DB8:100::2/64` | `2001:DB8:100::/64` |
| R2     | G0/1      | `2001:DB8:13::1/64`  | `2001:DB8:13::/64`  |
| PC1    | Fa0/0     | `2001:DB8:13::10/64` | `2001:DB8:13::/64`  |

### Default Gateways

* PC0 → `2001:DB8:12::1`
* PC1 → `2001:DB8:13::1`

---

## Important IPv6 Terminology

### IPv6

IPv6 is the successor to IPv4 and uses 128-bit addresses instead of IPv4's 32-bit addresses.

Example:

```text
2001:DB8:12::10
```

### `/64` Prefix

A `/64` means the first 64 bits identify the network prefix, while the remaining 64 bits identify the interface.

Example:

```text
2001:DB8:12::/64
```

### IPv6 Unicast Routing

IPv6 unicast routing allows a router to forward IPv6 packets between different IPv6 networks.

It is enabled with:

```text
ipv6 unicast-routing
```

### NDP — Neighbor Discovery Protocol

NDP is an IPv6 protocol that uses ICMPv6 to discover neighboring devices and perform functions that include address resolution and neighbor discovery.

It replaces the traditional ARP mechanism used by IPv4.

### Static IPv6 Route

A static route is a manually configured route telling a router where to forward traffic destined for a particular network.

---

# Router Configuration

## R1

### Enable IPv6 Routing

```text
enable
configure terminal
ipv6 unicast-routing
```

### LAN A

```text
interface gigabitEthernet 0/0
ipv6 address 2001:DB8:12::1/64
no shutdown
```

### R1-R2 Link

```text
interface gigabitEthernet 0/1
ipv6 address 2001:DB8:100::1/64
no shutdown
```

### Static Route to LAN B

```text
ipv6 route 2001:DB8:13::/64 2001:DB8:100::2
```

---

## R2

### Enable IPv6 Routing

```text
enable
configure terminal
ipv6 unicast-routing
```

### R1-R2 Link

```text
interface gigabitEthernet 0/0
ipv6 address 2001:DB8:100::2/64
no shutdown
```

### LAN B

```text
interface gigabitEthernet 0/1
ipv6 address 2001:DB8:13::1/64
no shutdown
```

### Static Return Route to LAN A

```text
ipv6 route 2001:DB8:12::/64 2001:DB8:100::1
```

---

# Routing Tables

R1 contained:

```text
C  2001:DB8:12::/64
C  2001:DB8:100::/64
S  2001:DB8:13::/64
```

R2 contained:

```text
C  2001:DB8:100::/64
C  2001:DB8:13::/64
S  2001:DB8:12::/64
```

Where:

* `C` = Connected network
* `S` = Static route

The static routes provide connectivity between the two LANs.

---

# NDP Verification

NDP was verified using:

```text
show ipv6 neighbors
```

Both R1 and R2 successfully learned IPv6 neighbors.

This confirmed that IPv6 Neighbor Discovery was functioning on the network.

---

# Troubleshooting

Initially, the following test failed:

```text
PC0 → PC1
```

The packet simulation showed that the packet failed at R1.

The troubleshooting process was performed systematically.

### 1. Checked R1's routing table

The route to:

```text
2001:DB8:13::/64
```

was present.

### 2. Tested R1 → R2

```text
ping 2001:DB8:100::2
```

Result:

```text
Successful
```

### 3. Tested R2 → PC1

```text
ping 2001:DB8:13::10
```

Result:

```text
Successful
```

### 4. Checked R2's return route

The route to:

```text
2001:DB8:12::/64
```

was present.

### 5. Checked IPv6 unicast routing

On R1:

```text
show running-config | include ipv6 unicast-routing
```

No output was returned.

This identified the problem.

### 6. Enabled IPv6 unicast routing

```text
configure terminal
ipv6 unicast-routing
end
```

After enabling IPv6 forwarding, R1 could successfully forward IPv6 traffic.

---

# Connectivity Testing

After the configuration was corrected:

### R1 → R2

```text
ping 2001:DB8:100::2
```

Successful.

### R2 → PC1

```text
ping 2001:DB8:13::10
```

Successful.

### R1 → PC1

```text
ping 2001:DB8:13::10
```

Successful.

### PC0 → PC1

```text
ping 2001:DB8:13::10
```

Result:

```text
Success rate is 100 percent (4/4)
```

This confirmed successful end-to-end IPv6 connectivity.

---

# Key Lesson

A router can have:

* Correct IPv6 addresses
* Correct interfaces
* Correct static routes
* Reachable next-hop routers

and still fail to forward IPv6 traffic if **IPv6 unicast routing is not enabled**.

The command:

```text
ipv6 unicast-routing
```

is therefore an important part of configuring a Cisco router for IPv6 routing.

---

# Cybersecurity Relevance

Understanding IPv6 is important for cybersecurity because modern networks can operate with both IPv4 and IPv6.

Security testing and network reconnaissance may encounter:

* IPv4 networks
* IPv6 networks
* Dual-stack networks
* IPv6-only segments
* IPv6 routing
* ICMPv6
* NDP

A security professional who only understands IPv4 can miss hosts, services, or network paths operating through IPv6.

Understanding routing is also important during penetration testing because connectivity problems can originate from routing, ACLs, firewalls, host configuration, or services.

---

# Skills Demonstrated

* IPv6 addressing
* IPv6 subnetting
* Cisco IOS configuration
* IPv6 interface configuration
* IPv6 unicast routing
* Static IPv6 routing
* Routing table analysis
* NDP verification
* IPv6 troubleshooting
* Packet-flow analysis
* End-to-end connectivity testing

---

# Lab Result

**Status: COMPLETE ✅**

Successful end-to-end communication was established between:

```text
PC0
2001:DB8:12::10
       ↓
     R1
       ↓
     R2
       ↓
PC1
2001:DB8:13::10
```

Final connectivity test:

```text
4/4 packets successful
```

---

## What I Learned

In this lab I learned how IPv6 addressing and routing work across multiple networks. I configured IPv6 interfaces, enabled IPv6 unicast routing, configured static routes, verified NDP neighbors, examined IPv6 routing tables, and troubleshot a failed connection.

The most important troubleshooting lesson was that having a correct routing table does not necessarily mean the router is capable of forwarding IPv6 traffic. IPv6 unicast routing must be enabled for the router to forward IPv6 packets between interfaces.
