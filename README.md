# 🌐 OSPF LSA Types 1-5 — Multi-Area OSPF with ABR & ASBR

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![OSPF](https://img.shields.io/badge/Protocol-OSPF-teal?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%2FCCNP-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Focus-LSA%20Types%201--5-purple?style=for-the-badge)

> A comprehensive Cisco Packet Tracer lab building a full multi-area OSPF topology — complete with an Area Border Router (ABR) and an Autonomous System Boundary Router (ASBR) — to capture and document **all five core OSPF LSA types** with real, verifiable CLI evidence.

---

## 📖 Overview

OSPF's Link-State Advertisements (LSAs) are usually taught as an abstract list of numbers and names. This lab makes them concrete: a real two-area topology with a genuine ABR and ASBR, where every LSA type can be directly observed in `show ip ospf database`, and every resulting routing table entry can be traced back to the exact LSA type that produced it.

**Goals of this lab:**
- Build a working multi-area OSPF topology (Area 0 + Area 1) with a real ABR and ASBR
- Capture and identify Type 1 through Type 5 LSAs with direct CLI evidence
- Confirm ABR and ASBR roles using `show ip ospf`
- Cross-reference routing table codes (`O`, `O IA`, `O E2`) back to their originating LSA type

---

## 🗺️ Network Topology

```
   L1: 10.1.1.1/32                L1: 20.1.1.1/32                L1: 30.1.1.1/32
                                                                  (ASBR — redistributes
                                                                   4 static Null0 routes)

[PC0-2] 192.168.1.x ─┐                                                          ┌─ 192.168.3.x [PC6-8]
                     ├─[Switch0]──[  R1  ]══1.1.1.0/24══[  R2  ]══2.1.1.0/24══[  R3  ]──[Switch2]─┤
                     ┘   Gi0/0      2911    S0/0/0        2911     g0/1/fg0/1   2911    Gi0/0      ┘
                     192.168.1.1/24                          │
                        AREA 0                            Gi0/0 192.168.2.1/24        AREA 1
                                                              AREA 0

                           ◄──────────── AREA 0 ────────────►│◄──────── AREA 1 ────────►
                                                            R2 = ABR                R3 = ASBR
```

| Device | Role                                                                  |
|--------|----------------------------------------------------------------------|
| R1     | Internal router — entirely within Area 0                             |
| R2     | **Area Border Router (ABR)** — sits on the boundary between Area 0 and Area 1 |
| R3     | **Autonomous System Boundary Router (ASBR)** — redistributes 4 static routes into OSPF |

---

## 🧾 IP Addressing & Area Table

| Device | Interface   | IP Address     | Area    | Purpose                     |
|--------|-------------|-----------------|-----------|--------------------------------|
| R1     | Gi0/0       | 192.168.1.1     | Area 0    | LAN1                            |
| R1     | S0/0/0      | 1.1.1.1         | Area 0    | To R2                           |
| R1     | Loopback1   | 10.1.1.1        | Area 0    | Router ID                       |
| R2     | S0/0/0      | 1.1.1.2         | Area 0    | To R1                           |
| R2     | Gi0/0       | 192.168.2.1     | Area 0    | LAN2                            |
| R2     | g0/1        | 2.1.1.1         | Area 1    | To R3                           |
| R2     | Loopback1   | 20.1.1.1        | Area 0    | Router ID (ABR)                 |
| R3     | fg0/1       | 2.1.1.2         | Area 1    | To R2                           |
| R3     | Gi0/0       | 192.168.3.1     | Area 1    | LAN3                            |
| R3     | Loopback1   | 30.1.1.1        | Area 1    | Router ID (ASBR)                |

### Redistributed static routes on R3 (making it an ASBR)
```
R3(config)#ip route 192.1.65.0 255.255.255.0 Null0
R3(config)#ip route 192.1.66.0 255.255.255.0 Null0
R3(config)#ip route 192.1.67.0 255.255.255.0 Null0
R3(config)#ip route 192.1.68.0 255.255.255.0 Null0

R3(config)#router ospf 1
R3(config-router)#redistribute static subnets
```

---

## 🔢 The 5 Core OSPF LSA Types — Quick Reference

| Type | Name                    | Originated by                | Scope / Purpose                                              |
|------|--------------------------|-------------------------------|------------------------------------------------------------------|
| 1    | Router LSA               | Every OSPF router              | Describes the router's own links, flooded within its area only |
| 2    | Network LSA              | The DR of a multi-access segment | Describes all routers attached to that segment                |
| 3    | Summary LSA (Inter-Area) | ABR                            | Advertises a network from one area into another                |
| 4    | ASBR Summary LSA         | ABR                            | Advertises "an ASBR exists at this Router ID" into other areas |
| 5    | AS External LSA          | ASBR                           | Advertises externally redistributed routes across all areas     |

---

## 🔍 Live Evidence — Captured Per LSA Type

### 1️⃣ & 2️⃣ Type 1 (Router) & Type 2 (Network) LSAs — `show ip ospf database` on R2 (ABR)
```
R2#show ip ospf database
OSPF Router with ID (20.1.1.1) (Process ID 1)

                Router Link States (Area 0)
Link ID         ADV Router      Age    Seq#       Checksum Link count
10.1.1.1        10.1.1.1        803    0x80000004 0x00b33e 4
20.1.1.1        20.1.1.1        134    0x80000004 0x009d65 3

                Router Link States (Area 1)
Link ID         ADV Router      Age    Seq#       Checksum Link count
192.168.2.1     192.168.2.1     616    0x80000003 0x00fd4a 2
30.1.1.1        30.1.1.1        133    0x80000006 0x0090c1 3
20.1.1.1        20.1.1.1        64     0x80000005 0x00b13d 2

                Net Link States (Area 1)
Link ID         ADV Router      Age    Seq#       Checksum
2.1.1.2         30.1.1.1        64     0x80000002 0x00f8a1
```
> **Type 1 (Router LSA):** each router (R1, R2, R3) originates its own Router LSA in every area it participates in, describing its own directly connected links — visible per-area (Area 0 and Area 1 have separate Router Link States sections).
>
> **Type 2 (Network LSA):** originated by the segment's **DR** — here, `30.1.1.1` (R3) is DR on the R2–R3 link, so it generates the Net Link State for `2.1.1.2`.

### 3️⃣ Type 3 — Summary LSA (Inter-Area)
```
                Summary Net Link States (Area 0)
Link ID         ADV Router      Age    Seq#       Checksum
2.1.1.0         20.1.1.1        124    0x80000001 0x0098aa
20.1.1.1        20.1.1.1        111    0x80000004 0x009d8f
30.1.1.1        20.1.1.1        59     0x80000005 0x0023fd
192.168.3.0     20.1.1.1        59     0x80000006 0x00f6dd

                Summary Net Link States (Area 1)
Link ID         ADV Router      Age    Seq#       Checksum
1.1.1.0         20.1.1.1        129    0x80000001 0x001ee6
192.168.2.0     20.1.1.1        129    0x80000002 0x00ffda
192.168.1.0     20.1.1.1        129    0x80000003 0x008b0f
10.1.1.1        20.1.1.1        129    0x80000004 0x00a353
```
> **Confirmed:** every single Summary LSA in both areas is advertised by `20.1.1.1` — R2, the ABR. R2 takes networks that live in Area 1 (like `192.168.3.0` and `30.1.1.1`) and re-advertises them into Area 0 as Type 3 Summary LSAs, and does the exact same thing in reverse for Area 0's networks into Area 1. **This is the defining job of an ABR.**

### 4️⃣ Type 4 — ASBR Summary LSA — `show ip ospf database` on R1
```
                Summary ASB Link States (Area 0)
Link ID         ADV Router      Age    Seq#       Checksum
30.1.1.1        20.1.1.1        230    0x80000009 0x00031a
```
> **Confirmed:** R2 (the ABR) tells Area 0 "the ASBR with Router ID `30.1.1.1` (R3) exists, and here's how to reach it." Without this Type 4 LSA, routers in Area 0 would have no way to know how to forward traffic toward R3's external routes, even after receiving the Type 5 LSAs themselves.

### 5️⃣ Type 5 — AS External LSA
```
                Type-5 AS External Link States
Link ID         ADV Router      Age    Seq#       Checksum Tag
192.1.65.0      30.1.1.1        235    0x80000003 0x00fd9c 0
192.1.66.0      30.1.1.1        235    0x80000003 0x00f2a6 0
192.1.67.0      30.1.1.1        235    0x80000003 0x00e7b0 0
192.1.68.0      30.1.1.1        235    0x80000003 0x00dcba 0
```
> **Confirmed:** all four of R3's redistributed static routes appear here, all advertised by `30.1.1.1` (R3, the ASBR). Type 5 LSAs are the only LSA type that flood **unchanged across every area** in the OSPF domain — they are never summarized or altered by an ABR the way Type 1/2 LSAs are.

---

## 🧭 Confirming ABR and ASBR Roles Directly

### R2 — confirmed as ABR
```
R2#show ip ospf
Routing Process "ospf 1" with ID 20.1.1.1
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
    Area BACKBONE(0)
      Number of interfaces in this area is 2
    Area 1
      Number of interfaces in this area is 2
```

### R3 — confirmed as ASBR
```
R3#show ip ospf
Routing Process "ospf 1" with ID 30.1.1.1
  It is an autonomous system boundary router
  Number of external LSA 4. Checksum Sum 0x03b4ac
    Area 1
      Number of interfaces in this area is 3
```

### R1 — confirmed as a plain internal router (neither ABR nor ASBR)
```
R1#show ip ospf
Routing Process "ospf 1" with ID 10.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
    Area BACKBONE(0)
      Number of interfaces in this area is 3
```
> R1's output contains **no** "area border router" or "autonomous system boundary router" line — confirming it's a standard internal router, participating in exactly one area.

---

## 📊 Routing Table Cross-Reference — R1's `show ip route`

```
O IA  2.1.1.0/24    [110/65] via 1.1.1.2, Serial0/0/0        ← Type 3 (inter-area)
O IA  20.1.1.1/32   [110/65] via 1.1.1.2, Serial0/0/0        ← Type 3 (inter-area)
O IA  30.1.1.1/32   [110/66] via 1.1.1.2, Serial0/0/0        ← Type 3 (inter-area)
O E2  192.1.65.0/24 [110/20] via 1.1.1.2, Serial0/0/0        ← Type 4 + Type 5 (external)
O E2  192.1.66.0/24 [110/20] via 1.1.1.2, Serial0/0/0        ← Type 4 + Type 5 (external)
O E2  192.1.67.0/24 [110/20] via 1.1.1.2, Serial0/0/0        ← Type 4 + Type 5 (external)
O E2  192.1.68.0/24 [110/20] via 1.1.1.2, Serial0/0/0        ← Type 4 + Type 5 (external)
O     192.168.2.0/24 [110/65] via 1.1.1.2, Serial0/0/0       ← Type 1 + Type 2 (intra-area)
O IA  192.168.3.0/24 [110/66] via 1.1.1.2, Serial0/0/0       ← Type 3 (inter-area)
```

| Route Code | Meaning                       | LSA Type(s) Behind It          |
|------------|--------------------------------|-----------------------------------|
| `O`        | Intra-area route               | Type 1 (Router) + Type 2 (Network) |
| `O IA`     | Inter-area route                | Type 3 (Summary)                  |
| `O E2`     | External Type 2 route           | Type 4 (ASBR Summary) + Type 5 (External) working together |

> **This is the full picture:** every routing table entry OSPF ever produces traces directly back to one or more of these five LSA types — nothing in the table is arbitrary.

---

## 🎯 Key Learnings

- **Type 1 (Router)** and **Type 2 (Network)** LSAs stay confined to a single area — they are never seen outside the area they were originated in.
- **Type 3 (Summary)** LSAs are exclusively originated by **ABRs** — they are the mechanism that carries reachability information between areas without flooding the full topology across the boundary.
- **Type 4 (ASBR Summary)** LSAs solve a specific problem: without them, routers in a different area would receive Type 5 external routes but have no path information to reach the ASBR that originated them. The ABR fixes this by explicitly advertising the ASBR's location.
- **Type 5 (AS External)** LSAs are the only LSA type that floods completely unchanged across every area in the OSPF domain — an ABR never summarizes or alters them.
- The routing table codes `O`, `O IA`, and `O E2` are not arbitrary labels — each one maps directly and predictably back to specific LSA types, which is genuinely useful for fast troubleshooting.
- `show ip ospf` directly states whether a router is an ABR (`"It is an area border router"`) or ASBR (`"It is an autonomous system boundary router"`) — no guesswork required.

---

## ✅ Outcomes

- Built and fully documented a working multi-area OSPF topology with a genuine ABR and ASBR
- Captured direct CLI evidence for all 5 core OSPF LSA types
- Confirmed ABR and ASBR roles using `show ip ospf`
- Cross-referenced every routing table entry back to its originating LSA type
- Produced the most advanced and complete OSPF reference lab in this series — genuinely useful study material for CCNA and early CCNP-level OSPF concepts

---

## 🛠️ Tools Used

- Cisco Packet Tracer
- Cisco IOS (Multi-Area OSPF, Static Route Redistribution)
- `show ip ospf database`, `show ip ospf`, `show ip route`

---

## 👤 Author

**Maaz Khan**
CCNA Certified | Network & NOC Engineer
📍 Lower Dir, KPK, Pakistan
🔗 [LinkedIn](https://www.linkedin.com/in/maazkhanms) · [GitHub](https://github.com/maazkhanms)

---

⭐ If you found this lab useful, consider starring the repo — more RIP, OSPF, EIGRP, IPv6, and routing-fundamentals labs coming in this series!
