# Lab 05 — Access Control Lists (ACLs)

## Overview

This lab demonstrates how Cisco Access Control Lists (ACLs) can be used to control network traffic based on defined security rules.

The lab uses an **Extended ACL** to block ICMP traffic from one specific host to another while allowing other IPv4 traffic.

---

## Objectives

* Understand the purpose of Access Control Lists (ACLs)
* Configure an Extended ACL
* Understand ACEs and ACL rule ordering
* Control traffic using source and destination IP addresses
* Understand ICMP filtering
* Apply an ACL to a router interface
* Understand inbound and outbound ACL direction
* Verify ACL operation using connectivity tests and hit counts

---

## Topology

```text
                 Router R0
              ┌─────────────┐
              │ Cisco 1941  │
              └─────┬───┬───┘
                  G0/0  G0/1
                    │    │
              ┌─────┘    └─────┐
              │                │
          Switch A          Switch B
          /  |  \            /  \
        PC0 PC1 PC2         PC3 PC4
```

### Devices

* 1 × Cisco 1941 Router
* 2 × Cisco 2960 Switches
* 5 × PCs
* Copper Straight-Through cables

---

## Network Design

### Network A

**Network:** `192.168.50.0/24`

| Device      | IP Address      | Subnet Mask     | Default Gateway |
| ----------- | --------------- | --------------- | --------------- |
| Router G0/0 | `192.168.50.1`  | `255.255.255.0` | —               |
| PC0         | `192.168.50.10` | `255.255.255.0` | `192.168.50.1`  |
| PC1         | `192.168.50.20` | `255.255.255.0` | `192.168.50.1`  |
| PC2         | `192.168.50.30` | `255.255.255.0` | `192.168.50.1`  |

### Network B

**Network:** `192.168.60.0/24`

| Device      | IP Address      | Subnet Mask     | Default Gateway |
| ----------- | --------------- | --------------- | --------------- |
| Router G0/1 | `192.168.60.1`  | `255.255.255.0` | —               |
| PC3         | `192.168.60.10` | `255.255.255.0` | `192.168.60.1`  |
| PC4         | `192.168.60.20` | `255.255.255.0` | `192.168.60.1`  |

---

## Router Configuration

### G0/0 — Network A

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.50.1 255.255.255.0
no shutdown
exit
```

### G0/1 — Network B

```text
interface gigabitEthernet 0/1
ip address 192.168.60.1 255.255.255.0
no shutdown
exit
```

Both interfaces were verified as:

```text
up/up
```

---

## ACL Configuration

A named Extended ACL was created:

```text
ip access-list extended BLOCK-PC0-TO-PC3
deny icmp host 192.168.50.10 host 192.168.60.10
permit ip any any
exit
```

### ACL Rules

**Rule 10:**

```text
deny icmp host 192.168.50.10 host 192.168.60.10
```

Blocks ICMP traffic from **PC0** to **PC3**.

**Rule 20:**

```text
permit ip any any
```

Allows other IPv4 traffic.

---

## Applying the ACL

The ACL was applied outbound on Router R1's G0/1 interface:

```text
interface gigabitEthernet 0/1
ip access-group BLOCK-PC0-TO-PC3 out
exit
```

The configuration was verified using:

```text
show ip interface gigabitEthernet 0/1
```

The router confirmed:

```text
Outgoing access list is BLOCK-PC0-TO-PC3
```

---

## Testing

### Before ACL

PC0 could successfully communicate with PC3 and PC4.

```text
PC0 → PC3    Successful
PC0 → PC4    Successful
```

This established the baseline before applying the security policy.

### After ACL

The ACL was then applied.

```text
PC0 → PC3    Blocked
PC0 → PC4    Successful
```

This demonstrated that the ACL was selectively filtering traffic rather than blocking PC0 from the entire network.

### ACL Verification

The ACL was verified using:

```text
show access-lists
```

The deny rule recorded matching traffic, confirming that the ACL was actively processing packets.

---

## Key Terminology

### ACL — Access Control List

A collection of rules used to permit or deny network traffic.

### ACE — Access Control Entry

An individual rule within an ACL.

### Extended ACL

An ACL capable of filtering traffic using information such as source IP, destination IP, protocol, and ports.

### ICMP

Internet Control Message Protocol. It is commonly used by network diagnostic tools such as `ping`.

### Inbound

Traffic entering a router interface.

### Outbound

Traffic leaving a router interface.

### Implicit Deny

An ACL concept where traffic that does not match a permit rule is ultimately denied.

### Hit Count

The number of times an ACL rule has matched traffic.

### Least Privilege

A security principle where systems and users receive only the access they require.

---

## Important Networking Concept

An ACL on a router can only inspect traffic that actually passes through that router interface.

Initially, all PCs were on the same subnet:

```text
PC0 ── Switch ── PC2
```

Their traffic could be switched directly without reaching the router.

After creating a second network:

```text
PC0
 ↓
Switch A
 ↓
R1
 ↓
Switch B
 ↓
PC3
```

traffic between the two networks had to pass through R1, allowing the ACL to inspect and filter it.

---

## Cybersecurity Relevance

ACLs are an important network security concept because they provide a method of controlling which traffic is allowed across network boundaries.

They can contribute to:

* Network segmentation
* Access control
* Traffic filtering
* Limiting attack paths
* Restricting access to sensitive systems
* Network defense
* Implementing least-privilege principles

ACLs also provide foundational knowledge for understanding firewall rules and other network security controls.

---

## Commands Practiced

```text
show ip interface brief
```

Used to verify router interface status.

```text
show ip interface gigabitEthernet 0/1
```

Used to verify ACL application and interface information.

```text
show access-lists
```

Used to view ACL rules and traffic match counts.

```text
ip access-list extended <name>
```

Used to create or enter a named Extended ACL.

```text
ip access-group <ACL-name> out
```

Used to apply an ACL to an interface in the outbound direction.

---

## What I Learned

* How ACLs control network traffic
* The difference between Standard and Extended ACLs
* How Extended ACLs can identify specific source and destination hosts
* How ICMP can be filtered
* How ACL direction affects traffic filtering
* How to apply an ACL to a router interface
* How to verify ACL configuration
* How ACL hit counts provide evidence of rule matches
* Why traffic must pass through the router for a router ACL to inspect it
* How network segmentation and access control contribute to cybersecurity

---

## Files

* `access-control-lists.pkt` — Cisco Packet Tracer project
* `topology.png` — Network topology diagram
* `README.md` — Lab documentation

---

## Status

**Completed **

This lab demonstrates practical experience with Cisco routing, Extended ACL configuration, traffic filtering, network segmentation, and basic network security controls.
