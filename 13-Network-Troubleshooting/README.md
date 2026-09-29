# Lab 13 — Network Troubleshooting

## Overview

This lab focuses on identifying, diagnosing, and resolving common network connectivity problems using Cisco Packet Tracer.

Instead of simply configuring a working network, faults were deliberately introduced and investigated using a structured troubleshooting methodology.

The lab covered endpoint addressing, routing, interface status, default gateways, and end-to-end connectivity.

---

## Objectives

* Develop a structured network troubleshooting methodology
* Diagnose incorrect IPv4 addressing
* Diagnose missing routing information
* Understand bidirectional routing
* Diagnose interface failures
* Understand the role of the default gateway
* Use Cisco IOS troubleshooting commands
* Verify connectivity before and after troubleshooting
* Document the cause, diagnosis, and resolution of network faults

---

# Topology

```text
PC2
 |
 | Fa0
 |
Switch3
 |
 | Fa0/24
 |
Router3
 | G0/1
 |
 | 10.0.0.0/30
 |
 | G0/0
Router4
 |
 | G0/1
 |
Switch4
 |
 | Fa0
 |
PC3
```

---

# Device Roles

| Role         | Packet Tracer Device |
| ------------ | -------------------- |
| Left host    | PC2                  |
| Left switch  | Switch3              |
| Left router  | Router3              |
| Right router | Router4              |
| Right switch | Switch4              |
| Right host   | PC3                  |

---

# IPv4 Addressing Plan

| Device  | Interface | IPv4 Address    | Subnet Mask       | Default Gateway |
| ------- | --------- | --------------- | ----------------- | --------------- |
| PC2     | Fa0       | `192.168.10.10` | `255.255.255.0`   | `192.168.10.1`  |
| Router3 | G0/0      | `192.168.10.1`  | `255.255.255.0`   | —               |
| Router3 | G0/1      | `10.0.0.1`      | `255.255.255.252` | —               |
| Router4 | G0/0      | `10.0.0.2`      | `255.255.255.252` | —               |
| Router4 | G0/1      | `192.168.20.1`  | `255.255.255.0`   | —               |
| PC3     | Fa0       | `192.168.20.10` | `255.255.255.0`   | `192.168.20.1`  |

---

# Network Structure

The network contains three IPv4 networks:

### LAN A

```text
192.168.10.0/24
```

### Router-to-Router Network

```text
10.0.0.0/30
```

### LAN B

```text
192.168.20.0/24
```

---

# Troubleshooting Methodology

The following troubleshooting process was used:

```text
Identify the symptom
        ↓
Test connectivity
        ↓
Localize the failure
        ↓
Inspect configuration
        ↓
Identify the root cause
        ↓
Apply the correction
        ↓
Retest
        ↓
Document the result
```

This prevents random configuration changes and allows the problem to be isolated systematically.

---

# Important Troubleshooting Commands

## `ping`

Used to test IP connectivity between two devices.

Example:

```text
ping 192.168.20.1
```

A successful ping indicates that ICMP traffic can reach the destination and receive a response.

---

## `show ip interface brief`

Used to quickly examine router interfaces.

Example:

```text
show ip interface brief
```

Important fields include:

* Interface
* IP address
* Status
* Protocol

An operational interface normally appears as:

```text
up    up
```

---

## `show ip route`

Displays the router's IPv4 routing table.

Example:

```text
show ip route
```

This allows the administrator to determine whether the router knows how to reach a destination network.

---

## `show interfaces`

Provides detailed information about an interface, including operational status and interface statistics.

---

# Fault 1 — Incorrect IP Address

## Symptom

PC3 could not communicate with its default gateway.

Initial testing:

```text
PC3 → 192.168.20.1
```

Result:

```text
4 sent, 4 lost
```

## Investigation

PC3 was found to have an incorrect IP address beginning with:

```text
198.168.20.x
```

Instead of:

```text
192.168.20.10
```

## Root Cause

PC3 was configured for the wrong IPv4 network.

Router4's LAN interface was:

```text
192.168.20.1/24
```

Therefore PC3 needed to belong to:

```text
192.168.20.0/24
```

## Resolution

PC3 was corrected to:

```text
IP address:      192.168.20.10
Subnet mask:     255.255.255.0
Default gateway: 192.168.20.1
```

## Verification

```text
ping 192.168.20.1
```

Result:

```text
4/4 successful
```

---

# Fault 2 — Missing Route on Router3

After correcting PC3, the local networks were tested.

PC2 could reach Router3.

PC3 could reach Router4.

Router3 could reach Router4:

```text
ping 10.0.0.2
```

However:

```text
PC2 → PC3
```

failed.

## Investigation

Router3's routing table was inspected:

```text
show ip route
```

There was no route to:

```text
192.168.20.0/24
```

## Root Cause

Router3 did not know where to send traffic destined for PC3's network.

## Resolution

A static route was configured:

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

This tells Router3:

> To reach `192.168.20.0/24`, forward traffic to Router4 at `10.0.0.2`.

The route appeared as:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

---

# Fault 3 — Missing Return Route on Router4

After configuring Router3, Router4 was checked.

Router4 did not have a route to:

```text
192.168.10.0/24
```

## Root Cause

Routing needs to work in both directions.

Router3 knew:

```text
192.168.20.0/24 → 10.0.0.2
```

But Router4 did not know:

```text
192.168.10.0/24 → 10.0.0.1
```

Without the return route, responses could not properly return to PC2.

## Resolution

The following static route was configured on Router4:

```text
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

The route then appeared as:

```text
S 192.168.10.0/24 [1/0] via 10.0.0.1
```

---

# Fault 4 — Interface Shutdown

A fourth fault was deliberately introduced on Router4.

The LAN interface was administratively disabled:

```text
interface gigabitEthernet 0/1
shutdown
```

## Symptom

PC3 attempted to reach its gateway:

```text
ping 192.168.20.1
```

Result:

```text
4 sent, 4 lost
```

## Investigation

Router4 was checked with:

```text
show ip interface brief
```

G0/1 showed:

```text
down    down
```

## Root Cause

The interface had been administratively disabled.

The `shutdown` command turns an interface off.

## Resolution

The interface was restored:

```text
interface gigabitEthernet 0/1
no shutdown
```

The interface returned to:

```text
up    up
```

## Verification

PC3 successfully reached its gateway:

```text
ping 192.168.20.1
```

Result:

```text
4/4 successful
```

---

# Fault 5 — Incorrect Default Gateway

The final troubleshooting scenario involved an incorrect default gateway.

PC3's gateway was deliberately changed from:

```text
192.168.20.1
```

to:

```text
192.168.20.254
```

## Local Network Test

PC3 could still ping:

```text
192.168.20.1
```

This was expected because both devices belong to:

```text
192.168.20.0/24
```

The default gateway is not required when communicating directly with another host on the same subnet.

## Remote Network Test

PC3 attempted to reach:

```text
192.168.10.10
```

The test failed.

## Root Cause

`192.168.10.0/24` is a different network.

PC3 therefore needed to forward the packet to its default gateway.

Because the gateway was incorrectly configured as:

```text
192.168.20.254
```

the packet could not be properly forwarded.

## Resolution

The gateway was restored to:

```text
192.168.20.1
```

## Verification

PC3 successfully reached PC2:

```text
ping 192.168.10.10
```

Result:

```text
4/4 successful
```

---

# Final Routing Configuration

## Router3

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

## Router4

```text
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

This creates bidirectional routing between the two LANs.

---

# Final Connectivity

After all faults were resolved:

```text
PC2
 ↓
Switch3
 ↓
Router3
 ↓
Router4
 ↓
Switch4
 ↓
PC3
```

Final test:

```text
ping 192.168.20.10
```

Result:

```text
4 sent, 4 received
```

End-to-end connectivity was successfully restored.

---

# Key Concepts Learned

### IP Addressing

Hosts must have valid addresses belonging to the appropriate network.

### Subnet

A subnet defines which devices are considered part of the same IP network.

### Default Gateway

The gateway is used when a host needs to communicate with a different network.

### Routing

Routers use routing tables to determine where packets should be forwarded.

### Static Route

A manually configured route used to reach a remote network.

### Return Route

A route that allows response traffic to travel back toward the originating network.

### Interface Status

`up/up` generally indicates an operational interface, while `administratively down` indicates that it has been disabled through configuration.

### End-to-End Connectivity

Testing only individual links is not enough. The entire communication path must be tested.

---

# Cybersecurity Relevance

Network troubleshooting is an important foundation for cybersecurity and ethical hacking.

During security testing, a failed connection does not automatically indicate that a firewall or security control is blocking traffic.

The cause could be:

* Incorrect IP addressing
* Incorrect subnetting
* Missing routes
* Incorrect default gateway
* Interface failure
* VLAN configuration
* ACLs
* Firewall rules
* DNS problems
* Service availability

Understanding normal network behavior makes it easier to distinguish a genuine security control from a basic networking problem.

This is particularly important during reconnaissance and penetration testing because connectivity must be understood before interpreting scan results.

---

# Troubleshooting Skills Demonstrated

* IPv4 troubleshooting
* Endpoint configuration analysis
* Gateway verification
* Routing-table analysis
* Static route configuration
* Bidirectional routing
* Interface troubleshooting
* End-to-end connectivity testing
* Fault isolation
* Root-cause analysis
* Cisco IOS troubleshooting

---

# Final Result

**Lab 13 — Network Troubleshooting: COMPLETE ✅**

Five different network faults were identified, diagnosed, corrected, and verified.

The final network achieved successful end-to-end communication between:

```text
PC2 — 192.168.10.10
          ↓
      Router3
          ↓
      Router4
          ↓
PC3 — 192.168.20.10
```

Final connectivity:

```text
4/4 packets successful
```

---

## What I Learned

I learned how to troubleshoot network connectivity systematically instead of changing configurations randomly.

I practiced identifying problems involving IP addressing, routing, interface status, default gateways, and return paths.

The lab also demonstrated that successful local connectivity does not necessarily mean that remote network connectivity is working. Testing should therefore progress from the endpoint, to the local gateway, across the routed link, and finally to the remote destination.
