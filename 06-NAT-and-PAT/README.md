# Lab 06 — NAT & PAT

## Overview

This lab demonstrates **Network Address Translation (NAT)** and **Port Address Translation (PAT)** using Cisco Packet Tracer.

The objective was to configure an internal private network that communicates with an external network through an ISP router. PAT was configured on the internal router so that multiple internal hosts could share a single translated IP address.

---

## Objectives

* Configure an internal LAN.
* Configure a router-to-router WAN/transit connection.
* Configure an external network through an ISP router.
* Configure NAT inside and outside interfaces.
* Configure PAT using NAT overload.
* Use an ACL to identify the internal network for translation.
* Configure routing between the internal and external networks.
* Verify NAT/PAT using Cisco IOS commands.
* Analyse NAT translation entries and statistics.

---

## Topology

```text
PC0 ─┐
PC1 ─┼── Switch A ── R1 G0/0
PC2 ─┘                │
                      │ G0/1
                      │ 10.0.0.1
                      │
                  ISP G0/0
                  10.0.0.2
                  ISP Router
                      │
                  ISP G0/1
                 192.168.80.1
                      │
                   Switch B
                      │
                     PC3
```

---

## Network Addressing

| Device     | Interface | IP Address    | Subnet Mask     | Role             |
| ---------- | --------- | ------------- | --------------- | ---------------- |
| R1         | G0/0      | 192.168.70.1  | 255.255.255.0   | Internal gateway |
| R1         | G0/1      | 10.0.0.1      | 255.255.255.252 | WAN/transit      |
| ISP Router | G0/0      | 10.0.0.2      | 255.255.255.252 | WAN/transit      |
| ISP Router | G0/1      | 192.168.80.1  | 255.255.255.0   | External gateway |
| PC0        | NIC       | 192.168.70.10 | 255.255.255.0   | Internal host    |
| PC1        | NIC       | 192.168.70.20 | 255.255.255.0   | Internal host    |
| PC2        | NIC       | 192.168.70.30 | 255.255.255.0   | Internal host    |
| PC3        | NIC       | 192.168.80.10 | 255.255.255.0   | External host    |

---

## Key Terminology

### NAT — Network Address Translation

NAT translates an IP address from one address space to another.

In this lab, private internal addresses from `192.168.70.0/24` are translated when communicating toward the outside network.

### PAT — Port Address Translation

PAT is a form of NAT that allows **multiple internal hosts to share one translated IP address**.

It is commonly called **NAT overload** on Cisco devices.

In this lab:

```text
192.168.70.10 ─┐
192.168.70.20 ─┼──→ 10.0.0.1
192.168.70.30 ─┘
```

Multiple internal hosts are represented by R1's `10.0.0.1` address.

### Inside Local

The original address of the internal host.

Example:

```text
192.168.70.10
```

### Inside Global

The address representing the internal host after translation.

In this lab:

```text
10.0.0.1
```

### Outside Local

The outside host's address as seen from the internal router.

### Outside Global

The actual address of the outside host.

In this lab:

```text
192.168.80.10
```

### NAT Inside Interface

An interface connected toward the internal network.

R1:

```text
G0/0
```

### NAT Outside Interface

An interface connected toward the external/WAN network.

R1:

```text
G0/1
```

### ACL — Access Control List

An ACL is a set of rules used by a network device to identify or control traffic.

In this lab, the ACL was used by NAT to identify which internal addresses should be translated.

---

## R1 NAT Configuration

The internal interface was configured as the NAT inside interface:

```text
interface gigabitEthernet 0/0
ip nat inside
```

The WAN interface was configured as the NAT outside interface:

```text
interface gigabitEthernet 0/1
ip nat outside
```

The NAT ACL identifies the internal network:

```text
access-list 1 permit 192.168.70.0 0.0.0.255
```

PAT was configured using NAT overload:

```text
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

The `overload` keyword enables multiple internal hosts to share the address of the outside interface.

---

## Routing

R1 was configured with a default route toward the ISP router:

```text
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

This tells R1 to forward traffic for destinations it does not have a more specific route for to the ISP router.

---

## Verification

### Interface Verification

```text
show ip interface brief
```

R1 showed:

```text
GigabitEthernet0/0     192.168.70.1    up    up
GigabitEthernet0/1     10.0.0.1        up    up
```

This confirmed that both NAT interfaces were operational.

---

## PAT Connectivity Test

Internal hosts were tested against the external host:

```text
ping 192.168.80.10
```

The internal PCs successfully communicated with PC3.

---

## NAT Translation Verification

The command:

```text
show ip nat translations
```

was used to inspect active translations.

The following translations were observed:

```text
192.168.70.10 → 10.0.0.1
192.168.70.20 → 10.0.0.1
192.168.70.30 → 10.0.0.1
```

This demonstrated the main purpose of PAT:

> Multiple internal hosts were able to use the same translated address.

For example:

```text
Inside Local        Inside Global
192.168.70.10   →   10.0.0.1
192.168.70.20   →   10.0.0.1
192.168.70.30   →   10.0.0.1
```

The different ICMP identifiers shown in the translation table allowed the router to distinguish the different ICMP conversations.

---

## NAT Statistics

The command:

```text
show ip nat statistics
```

returned:

```text
Total translations: 0
Outside Interfaces: GigabitEthernet0/1
Inside Interfaces: GigabitEthernet0/0
Hits: 31
Misses: 60
Expired translations: 32
```

### Analysis

**Outside Interface**

```text
GigabitEthernet0/1
```

This confirms that R1's G0/1 is configured as the NAT outside interface.

**Inside Interface**

```text
GigabitEthernet0/0
```

This confirms that R1's G0/0 is configured as the NAT inside interface.

**Hits: 31**

NAT recorded 31 packets that matched existing translation entries.

**Misses: 60**

These represent packets that did not find an existing translation and required NAT processing.

**Expired translations: 32**

NAT translations are temporary. After the traffic stops, translations can expire and be removed.

Therefore:

```text
Total translations: 0
```

does **not** mean NAT was not working. The earlier `show ip nat translations` output directly demonstrated active translations, while the statistics showed that NAT had processed traffic.

---

## NAT/PAT Traffic Flow

The overall traffic flow was:

```text
PC0
192.168.70.10
      │
      ▼
Switch A
      │
      ▼
R1 G0/0
NAT INSIDE
      │
      ▼
PAT
      │
      ▼
R1 G0/1
10.0.0.1
NAT OUTSIDE
      │
      ▼
ISP Router
      │
      ▼
Switch B
      │
      ▼
PC3
192.168.80.10
```

---

## Security and Networking Relevance

NAT and PAT are important concepts in networking and cybersecurity because they affect how internal systems communicate with external networks.

Understanding NAT is useful when:

* Analysing network traffic.
* Interpreting packet captures.
* Understanding firewall and router configurations.
* Performing network reconnaissance.
* Troubleshooting connectivity.
* Understanding how private networks communicate through edge devices.
* Analysing logs where translated addresses appear instead of original internal addresses.

For cybersecurity professionals, understanding address translation is especially important when analysing network traffic and determining how an internal host's traffic appears after passing through a gateway.

---

## What I Learned

* How NAT translates IP addresses.
* How PAT allows multiple hosts to share one translated address.
* The difference between inside local and inside global addresses.
* How Cisco identifies NAT inside and outside interfaces.
* How an ACL can identify addresses for NAT translation.
* How to configure PAT using `overload`.
* How to verify translations using `show ip nat translations`.
* How to analyse NAT statistics.
* Why temporary NAT translations can expire.
* How routing and NAT work together.

---

## Verification Commands Used

```text
show ip interface brief
show ip nat translations
show ip nat statistics
show access-lists
show ip route
```

---

## Lab Result

**Status: Completed**

NAT/PAT was successfully configured and verified.

Multiple internal hosts from the `192.168.70.0/24` network successfully communicated with the external host `192.168.80.10`, while R1 translated their traffic through the same outside interface address, demonstrating PAT/NAT overload.
