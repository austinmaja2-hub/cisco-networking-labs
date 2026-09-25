# Lab 08 — Network Security

## Overview

This lab focuses on **Layer 2 network security** using Cisco Packet Tracer.

The lab demonstrates how switch security controls can help protect a network against unauthorized devices, rogue DHCP servers, and unauthorized access through unused switch ports.

### Security Controls Implemented

* Port Security
* Sticky MAC addresses
* Port Security violation/shutdown
* DHCP Snooping
* Trusted and untrusted DHCP ports
* Unused-port hardening
* Parking VLAN

---

## Topology

### Devices

* 1 × Cisco 2960 Switch
* 1 × Cisco 1941 Router
* 2 × PCs

### Connections

```text
                 Cisco 1941 Router
                       G0/0
                        |
                      Fa0/24
                        |
                 Cisco 2960 Switch
                   /           \
                Fa0/1         Fa0/2
                  |             |
                 PC0           PC1
```

Unused switch ports:

```text
Fa0/3 – Fa0/23
```

---

# Part 1 — Port Security

## Objective

Configure switch port security so that only an authorized device can use a specific switch port.

Port Security was configured on:

```text
Fa0/1
```

### Configuration

```text
enable
configure terminal
interface fastEthernet 0/1
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
end
```

### Configuration Explanation

**`switchport mode access`**

Forces the interface to operate as an access port.

**`switchport port-security`**

Enables MAC-based port security.

**`maximum 1`**

Allows only one secure MAC address on the port.

**`mac-address sticky`**

Allows the switch to dynamically learn the connected device's MAC address and associate it with the port.

**`violation shutdown`**

If an unauthorized MAC address is detected, the switch can place the interface into an error-disabled state.

---

## Verification

Command:

```text
show port-security interface fastEthernet 0/1
```

Final result:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Security Violation Count   : 0
```

The switch learned PC0's MAC address:

```text
0060.5C19.879D
```

as a sticky secure MAC address.

---

## Port Security Attack Simulation

An unauthorized-device scenario was simulated by moving another PC onto the protected port.

The switch detected the unauthorized MAC address and produced:

```text
Security violation count: 1
```

The interface entered:

```text
err-disabled
```

This demonstrated how Port Security can respond to an unauthorized device attempting to use a protected switch port.

The port was subsequently recovered using:

```text
enable
configure terminal
interface fastEthernet 0/1
shutdown
no shutdown
end
```

---

# Part 2 — DHCP Snooping

## Objective

Configure DHCP Snooping to help protect the network against unauthorized or rogue DHCP servers.

The DHCP server function was provided by the Cisco router.

Network:

```text
192.168.60.0/24
```

Router/DHCP gateway:

```text
192.168.60.1
```

### DHCP Snooping Configuration

```text
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 1
interface fastEthernet 0/24
ip dhcp snooping trust
exit
```

DHCP Option 82 was disabled because enabling it caused DHCP clients in this Packet Tracer simulation to fail DHCP renewal:

```text
no ip dhcp snooping information option
```

---

## Trusted and Untrusted Ports

Final verification:

```text
show ip dhcp snooping
```

Result:

```text
Interface                  Trusted
-----------------------    -------
FastEthernet0/1            no
FastEthernet0/2            no
FastEthernet0/24           yes
```

### Port Roles

| Port   | Role                  |
| ------ | --------------------- |
| Fa0/1  | Untrusted — PC0       |
| Fa0/2  | Untrusted — PC1       |
| Fa0/24 | Trusted — Router/DHCP |

The router-facing port is trusted because legitimate DHCP server traffic originates from the router.

The PC-facing ports remain untrusted.

---

# Part 3 — Unused-Port Security

## Objective

Unused switch ports should not remain active in the default VLAN.

The unused ports were identified using:

```text
show interfaces status
```

Unused ports:

```text
Fa0/3 – Fa0/23
```

A dedicated parking VLAN was created:

```text
vlan 999
name UNUSED-PORTS
```

The unused ports were assigned to VLAN 999 and administratively shut down:

```text
interface range fastEthernet 0/3 - 23
switchport mode access
switchport access vlan 999
shutdown
exit
```

---

## Verification

Command:

```text
show vlan brief
```

Result:

```text
999  UNUSED-PORTS  active
```

with:

```text
Fa0/3 – Fa0/23
```

assigned to the VLAN.

The ports were also verified as disabled:

```text
show interfaces status
```

Example:

```text
Fa0/3   disabled   999
Fa0/4   disabled   999
...
Fa0/23  disabled   999
```

### Why this matters

An unused active switch port can potentially provide network access if someone physically connects an unauthorized device.

Moving unused ports into a dedicated VLAN and shutting them down provides an additional layer of protection.

---

# Security Concepts Learned

## Port Security

A switch security feature that restricts which MAC addresses are allowed to use a switch port.

## MAC Address

A hardware/network interface address used for communication at Layer 2.

Example:

```text
0060.5C19.879D
```

## Sticky MAC

A dynamically learned MAC address that is associated with a secure switch port.

## Security Violation

Occurs when traffic is received from a MAC address that violates the configured Port Security policy.

## Err-Disabled

A switch interface state in which the interface has been automatically disabled because of a condition such as a security violation.

## DHCP Snooping

A Layer 2 security mechanism that identifies trusted and untrusted DHCP paths and helps prevent unauthorized DHCP servers from providing network configuration.

## Trusted Port

A switch port that is permitted to receive legitimate DHCP server responses.

## Untrusted Port

A switch port where DHCP server-originated traffic is not expected, normally a client-facing port.

## Parking VLAN

A dedicated VLAN used to isolate unused switch ports.

## Defense in Depth

Using multiple security controls instead of relying on a single security mechanism.

This lab demonstrates defense in depth through:

```text
Port Security
      +
DHCP Snooping
      +
Unused Port Shutdown
      +
Parking VLAN
```

---

# Verification Commands

The following commands were used to verify the security configuration:

```text
show port-security interface fastEthernet 0/1
```

```text
show vlan brief
```

```text
show interfaces status
```

```text
show ip dhcp snooping
```

---

# Packet Tracer Limitation

During testing, the DHCP Snooping binding table did not populate as expected:

```text
show ip dhcp snooping binding
```

returned:

```text
Total number of bindings: 0
```

DHCP clients were nevertheless able to obtain their addresses after DHCP Snooping was configured correctly and DHCP Option 82 was disabled.

This lab therefore documents the DHCP Snooping configuration and trusted/untrusted port verification without claiming that the binding table was successfully demonstrated.

---

# Security Outcome

The switch was hardened against several common Layer 2 threats and misconfigurations.

### Implemented

* Port Security on Fa0/1
* Sticky MAC learning
* Maximum of one MAC address
* Shutdown violation mode
* DHCP Snooping on VLAN 1
* Trusted DHCP uplink
* Untrusted client ports
* Dedicated unused-port VLAN
* Shutdown of unused switch ports

### Final Port Layout

```text
Fa0/1   → PC0       → VLAN 1 → Port Security
Fa0/2   → PC1       → VLAN 1 → DHCP Snooping Untrusted
Fa0/24  → Router    → VLAN 1 → DHCP Snooping Trusted

Fa0/3–Fa0/23        → VLAN 999 → Disabled
```

---

# Skills Demonstrated

* Cisco IOS configuration
* Layer 2 security
* Switch Port Security
* MAC address security
* DHCP Snooping
* VLAN security
* Access-port hardening
* Security verification
* Network troubleshooting
* Security attack simulation
* Defense-in-depth principles

---

## Evidence

Include the following screenshots in this repository:

1. Lab topology
2. Port Security configuration
3. Port Security verification showing the sticky MAC
4. Port Security violation / err-disabled state
5. DHCP Snooping verification
6. VLAN 999 configuration
7. `show interfaces status` showing unused ports disabled

---

## Lab Status

**Completed — Lab 08: Network Security** ✅
