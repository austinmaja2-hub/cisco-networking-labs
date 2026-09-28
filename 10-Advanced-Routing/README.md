# Lab 10 — Advanced Routing

## Overview

This lab focused on advanced dynamic routing concepts using Cisco Packet Tracer.

The practical implementation focused primarily on **OSPF (Open Shortest Path First)**, including neighbor relationships, route advertisement, path selection, OSPF cost, redundant links, and automatic failover.

Additional routing technologies and concepts including **EIGRP, BGP, Administrative Distance, routing metrics, static routing, default routes, and next-hop routing** were studied theoretically.

---

## Objectives

* Understand dynamic routing
* Configure and verify OSPF
* Understand OSPF areas and Router IDs
* Establish OSPF neighbor relationships
* Advertise networks using OSPF
* Understand OSPF cost and path selection
* Configure multiple paths between routers
* Demonstrate OSPF route selection
* Demonstrate routing failover
* Understand EIGRP and DUAL
* Understand BGP and Autonomous Systems
* Compare OSPF, EIGRP, and BGP
* Understand Administrative Distance and routing metrics
* Understand static and default routes

---

# 1. Topology

The practical topology consisted of two routers connected through two separate links.

```text
                 GigabitEthernet
        10.0.0.1/30 ↔ 10.0.0.2/30
             OSPF Cost: 100

     R1 =========================== R2
      |                               |
      |                               |
      |                               |
 LAN A                             LAN B
192.168.10.0/24                 192.168.20.0/24
      |                               |
     PC0                             PC1

                 Serial
        10.0.1.1/30 ↔ 10.0.1.2/30
             OSPF Cost: 64
```

### R1

| Interface | IP Address      | Purpose             |
| --------- | --------------- | ------------------- |
| G0/0      | 192.168.10.1/24 | LAN A               |
| G0/1      | 10.0.0.1/30     | Ethernet link to R2 |
| S0/1/0    | 10.0.1.1/30     | Serial link to R2   |

### R2

| Interface | IP Address      | Purpose             |
| --------- | --------------- | ------------------- |
| G0/0      | 10.0.0.2/30     | Ethernet link to R1 |
| G0/1      | 192.168.20.1/24 | LAN B               |
| S0/1/0    | 10.0.1.2/30     | Serial link to R1   |

### End Devices

| Device | IP Address       | Default Gateway |
| ------ | ---------------- | --------------- |
| PC0    | 192.168.10.10/24 | 192.168.10.1    |
| PC1    | 192.168.20.10/24 | 192.168.20.1    |

---

# 2. Key Terminology

## Dynamic Routing

Dynamic routing allows routers to exchange routing information automatically using routing protocols.

Instead of manually configuring every route, routers can learn routes from neighboring routers.

## Routing Protocol

A protocol used by routers to exchange information about reachable networks.

Examples:

* OSPF
* EIGRP
* BGP
* RIP

## OSPF

**OSPF = Open Shortest Path First**

OSPF is a dynamic **link-state routing protocol** that uses the Shortest Path First (SPF) algorithm to calculate routes.

## OSPF Area

An area is a logical grouping of OSPF networks and routers.

**Area 0** is the OSPF backbone area.

This lab used:

```text
Area 0
```

## OSPF Neighbor

An OSPF neighbor is another router with which an OSPF relationship has been established.

## OSPF Adjacency

The relationship formed between OSPF routers that allows them to exchange routing information.

A successful adjacency reaches:

```text
FULL
```

## Router ID

A unique identifier used by OSPF to identify a router.

In this lab:

```text
R1 = 10.0.1.1
R2 = 10.0.1.2
```

## LSA

**LSA = Link-State Advertisement**

LSAs contain information that OSPF routers use to build their link-state database and calculate routes.

## OSPF Cost

OSPF uses cost as its routing metric.

A lower total cost is preferred.

In this lab:

```text
Ethernet = 100
Serial   = 64
```

Therefore, OSPF preferred the Serial path when both paths were available.

---

# 3. OSPF Configuration

### R1

```text
router ospf 1
 network 10.0.1.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

### R2

```text
router ospf 1
 network 10.0.1.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

The `network` statements tell OSPF which connected networks should participate in the OSPF process.

---

# 4. Verifying OSPF Neighbors

The command:

```text
show ip ospf neighbor
```

was used to verify OSPF neighbor relationships.

The routers successfully established FULL adjacencies across both links.

Example:

```text
Neighbor ID     State
10.0.1.2        FULL
10.0.1.2        FULL
```

The two entries represented the two separate OSPF paths between R1 and R2.

---

# 5. OSPF Route Advertisement

R1 advertised:

```text
192.168.10.0/24
```

R2 advertised:

```text
192.168.20.0/24
```

This allowed each router to learn the remote LAN dynamically.

For example, R1 learned:

```text
O 192.168.20.0/24
```

The `O` indicates that the route was learned through OSPF.

---

# 6. Understanding the Routing Table

An example route was:

```text
O 192.168.20.0 [110/65] via 10.0.1.2, Serial0/1/0
```

This can be interpreted as:

| Value        | Meaning                 |
| ------------ | ----------------------- |
| O            | OSPF-learned route      |
| 192.168.20.0 | Destination network     |
| 110          | Administrative Distance |
| 65           | OSPF metric/cost        |
| 10.0.1.2     | Next-hop address        |
| Serial0/1/0  | Outgoing interface      |

---

# 7. OSPF Path Selection Experiment

The topology intentionally contained two paths between R1 and R2.

### Ethernet

```text
10.0.0.1 ↔ 10.0.0.2
```

OSPF cost:

```text
100
```

### Serial

```text
10.0.1.1 ↔ 10.0.1.2
```

OSPF cost:

```text
64
```

Because OSPF prefers the lower-cost path, the Serial connection became the preferred route.

The routing table showed:

```text
O 192.168.20.0 [110/65] via 10.0.1.2, Serial0/1/0
```

The total OSPF cost was:

```text
64 + 1 = 65
```

Therefore:

```text
65 < 100
```

and OSPF selected the Serial path.

---

# 8. OSPF Failover Experiment

The Serial interface was intentionally disabled to simulate a link failure:

```text
interface serial 0/1/0
shutdown
```

After OSPF detected the failure, the routing table changed and traffic moved to the Ethernet path.

No static route was required.

This demonstrated that OSPF can dynamically recalculate the available topology and select another valid path.

The Serial interface was then restored:

```text
interface serial 0/1/0
no shutdown
```

The OSPF neighbor relationship returned to:

```text
FULL
```

This demonstrated network redundancy and dynamic failover.

---

# 9. EIGRP — Theoretical Study

**EIGRP = Enhanced Interior Gateway Routing Protocol**

EIGRP is a Cisco-developed dynamic routing protocol commonly classified as an advanced distance-vector protocol.

EIGRP uses the:

**DUAL — Diffusing Update Algorithm**

DUAL helps EIGRP calculate loop-free paths and respond quickly to topology changes.

### EIGRP Tables

EIGRP maintains:

1. **Neighbor table** — EIGRP neighbors
2. **Topology table** — routes learned from neighbors
3. **Routing table** — best routes installed for forwarding

### Successor

The best route selected by EIGRP toward a destination.

### Feasible Successor

A qualifying backup route that can be used if the successor becomes unavailable.

### EIGRP Metric

The traditional EIGRP composite metric can consider:

* Bandwidth
* Delay
* Reliability
* Load

By default, bandwidth and delay are the primary metric components.

---

# 10. BGP — Theoretical Study

**BGP = Border Gateway Protocol**

BGP is primarily used for exchanging routing information between **Autonomous Systems (ASes)**.

It is fundamental to Internet-scale routing.

### Autonomous System

An Autonomous System is a collection of networks and routers under a common administrative authority that presents a common routing policy.

### eBGP

**External BGP**

Used between different Autonomous Systems.

```text
AS 65001
   |
  eBGP
   |
AS 65002
```

### iBGP

**Internal BGP**

Used between BGP routers inside the same Autonomous System.

---

# 11. OSPF vs EIGRP vs BGP

| Feature                | OSPF                | EIGRP                     | BGP                       |
| ---------------------- | ------------------- | ------------------------- | ------------------------- |
| Main purpose           | Internal routing    | Internal routing          | Inter-AS routing          |
| Type                   | Link-state          | Advanced distance-vector  | Path-vector               |
| Algorithm              | SPF                 | DUAL                      | Path selection process    |
| Metric                 | Cost                | Composite metric          | Path attributes           |
| Areas                  | Yes                 | No OSPF-style areas       | No                        |
| Neighbor relationships | Yes                 | Yes                       | Yes                       |
| Typical scope          | Enterprise networks | Mainly Cisco environments | Internet / large networks |

---

# 12. Administrative Distance

Administrative Distance (AD) is used by a router to determine which routing source is preferred when multiple sources provide a route to the same destination.

Common Cisco default values include:

| Route Source   | Administrative Distance |
| -------------- | ----------------------: |
| Connected      |                       0 |
| Static         |                       1 |
| EIGRP          |                      90 |
| OSPF           |                     110 |
| RIP            |                     120 |
| External EIGRP |                     170 |

Lower AD is preferred.

For example:

```text
EIGRP = 90
OSPF  = 110
```

If both provide a route to the same destination, EIGRP's route would normally be preferred because its AD is lower.

---

# 13. Administrative Distance vs Metric

These two concepts should not be confused.

### Administrative Distance

Determines which **routing source** is preferred.

### Metric

Determines which **path within a routing protocol** is preferred.

For example:

```text
EIGRP route → AD 90
OSPF route  → AD 110
```

Administrative Distance determines which protocol wins.

Within OSPF:

```text
Path A → Cost 10
Path B → Cost 50
```

OSPF selects Path A because it has the lower cost.

---

# 14. Static Routing

A static route is manually configured by an administrator.

Example:

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

This tells R1 to reach `192.168.20.0/24` through `10.0.0.2`.

Static routes do not automatically adapt to topology changes in the same way dynamic routing protocols do.

---

# 15. Default Route

A default route is:

```text
0.0.0.0/0
```

It represents destinations for which the router has no more specific route.

It is often used to forward traffic toward an ISP or upstream router.

Example:

```text
ip route 0.0.0.0 0.0.0.0 10.0.1.2
```

---

# 16. Next Hop

The next hop is the next router/interface to which a packet should be forwarded.

Example:

```text
O 192.168.20.0/24
  via 10.0.1.2
```

Here:

```text
10.0.1.2
```

is the next-hop address.

---

# 17. Verification Commands

The following commands were used or studied during the lab:

```text
show ip route
show ip route ospf
show ip ospf neighbor
show ip ospf interface
show ip ospf
show ip protocols
show ip interface
ping
```

These commands are useful for troubleshooting routing relationships, interfaces, routes, and connectivity.

---

# 18. Troubleshooting Experience

During the lab, an issue occurred where R1's `GigabitEthernet0/0` was physically operational but had:

```text
Internet protocol processing disabled
```

As a result, the `192.168.10.0/24` network was not being properly advertised through OSPF.

The interface was restored to Layer-3 operation with:

```text
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

After correcting the interface, OSPF was able to operate correctly with the LAN.

This reinforced an important troubleshooting principle:

> A physical interface being "up" does not necessarily mean Layer-3/IP routing is functioning correctly.

---

# 19. Key Lessons

This lab demonstrated that dynamic routing protocols allow networks to adapt to topology changes without requiring every route to be manually configured.

The practical OSPF experiments demonstrated:

* Neighbor formation
* Route advertisement
* OSPF areas
* Router IDs
* Routing metrics
* Multiple paths
* Path selection
* Redundancy
* Automatic failover
* Route recalculation
* Routing troubleshooting

The theoretical section introduced:

* EIGRP
* DUAL
* Successors
* Feasible successors
* BGP
* Autonomous Systems
* iBGP
* eBGP
* Administrative Distance
* Routing metrics
* Static routes
* Default routes
* Next-hop routing

---

# 20. Cybersecurity Relevance

Understanding routing is important for cybersecurity because network security depends heavily on understanding how traffic moves between systems.

Routing knowledge is directly relevant to:

* Network reconnaissance
* Nmap scanning
* Network segmentation
* Firewall placement
* ACLs
* Traffic analysis
* Network monitoring
* Attack-path analysis
* Lateral movement analysis
* Network troubleshooting
* Incident response

For example, during network reconnaissance, identifying a service is only part of the process. Understanding **which network the target belongs to, how traffic reaches it, and what routing boundaries exist** provides important context for security analysis.

---

# 21. Evidence

Recommended screenshots for this lab:

* OSPF neighbor table showing `FULL`
* OSPF routing table
* Initial Ethernet path selection
* Serial path selection after changing OSPF cost
* Routing table during failover
* OSPF neighbor returning to `FULL` after link restoration
* Packet Tracer topology

---

## Conclusion

Lab 10 demonstrated how dynamic routing protocols allow routers to learn network information, select paths, and respond to topology changes.

The practical OSPF implementation showed how routing metrics influence path selection and how redundant links can provide automatic failover.

The theoretical study of EIGRP and BGP provided a foundation for understanding additional routing protocols used in enterprise and Internet-scale networks.

**Status: Completed ✅**

**Next Lab: Lab 11 — Network Services**
