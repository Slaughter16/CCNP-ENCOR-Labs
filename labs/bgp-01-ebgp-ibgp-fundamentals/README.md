# Lab 04: iBGP Core with Dual Route Reflectors and Dual eBGP Upstreams

**ENCOR domain:** 3.0 Infrastructure
**Topics:** iBGP, route reflectors, peer-groups, OSPF underlay, eBGP, next-hop-self, best-path selection, redundancy
**Environment:** EVE-NG on Proxmox, Cisco IOS (version: \_\_\_)

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO (delete when done):\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Everything marked TODO is yours to fill in from your own lab. Replace the placeholders with real outputs, screenshots, and sentences about what you saw.

\---

## Objectives

1. Build an iBGP core (AS 65100) with two route reflectors using a peer-group.
2. Use OSPF as the underlay so iBGP peers can reach each other's loopbacks.
3. Peer eBGP over physical interface addresses toward two parallel upstream providers that converge on one destination AS.
4. Fix next-hop reachability on the route reflectors.
5. Prove that R1, R2, and R3 each see the destination prefix through two different AS-paths.
6. Fail one route reflector and prove the clients keep their routes.
7. Steer traffic toward one upstream with Local Preference and verify it with a traceroute.

## Findings

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Write 3 to 5 one-line results after you finish. Examples of the kind of line to write:
> - What happened to the clients' tables when one RR was shut down.
> - Which upstream path won by default, and which tiebreaker decided it.
> - What changed after the LocPref policy, with the traceroute to prove it.

\---

## Topology

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Add your EVE-NG topology screenshot here.
> `!\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\[Topology](screenshots/01-topology.png)`

```
                 AS 65100 (iBGP core, OSPF underlay)

        R1 ---------- R4 (RR) ---------- R6 (AS 65200) ----
        R2 ---------- R4 (RR)                                \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
        R3 ---------- R4 (RR)                                 R8 (AS 65400)
         \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\             |                                     /
          (each client also links to R5)                     /
        R1 ---------- R5 (RR) ---------- R7 (AS 65300) ----
        R2 ---------- R5 (RR)
        R3 ---------- R5 (RR)

        R4 <----------> R5   (RR to RR link)

        R8 originates 10.200.8.0/24 from Loopback100
```

## Addressing

All point-to-point links use /30 subnets. On each link, the lower-numbered router takes `.1` and the higher-numbered router takes `.2`. Every router has Loopback0 as `10.255.0.N/32`, where N is the router number, and that address is also its BGP router ID.

### Routers

|Router|AS|Role|Loopback0|Notes|
|-|-|-|-|-|
|R1|65100|iBGP client|10.255.0.1/32|peers with R4 and R5|
|R2|65100|iBGP client|10.255.0.2/32|peers with R4 and R5|
|R3|65100|iBGP client|10.255.0.3/32|peers with R4 and R5|
|R4|65100|route reflector|10.255.0.4/32|eBGP to R6|
|R5|65100|route reflector|10.255.0.5/32|eBGP to R7|
|R6|65200|upstream A|10.255.0.6/32|top path|
|R7|65300|upstream B|10.255.0.7/32|bottom path|
|R8|65400|destination|10.255.0.8/32|originates 10.200.8.0/24 (Loopback100)|

### Links

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Fill in the interface column from your own EVE-NG cabling.

|Link|Subnet|Left router (.1)|Right router (.2)|Interfaces|
|-|-|-|-|-|
|R1 to R4|10.10.14.0/30|R1|R4|TODO|
|R1 to R5|10.10.15.0/30|R1|R5|TODO|
|R2 to R4|10.10.24.0/30|R2|R4|TODO|
|R2 to R5|10.10.25.0/30|R2|R5|TODO|
|R3 to R4|10.10.34.0/30|R3|R4|TODO|
|R3 to R5|10.10.35.0/30|R3|R5|TODO|
|R4 to R5|10.10.45.0/30|R4|R5|TODO|
|R4 to R6|10.10.46.0/30|R4|R6|TODO|
|R5 to R7|10.10.57.0/30|R5|R7|TODO|
|R6 to R8|10.10.68.0/30|R6|R8|TODO|
|R7 to R8|10.10.78.0/30|R7|R8|TODO|

\---

## Design plan

|Item|Choice|
|-|-|
|IGP|OSPF process 100, area 0, on internal links and loopbacks of R1 to R5 only|
|eBGP links|R4-R6, R5-R7, R6-R8, R7-R8 stay out of OSPF|
|iBGP peering|Loopback to loopback, `update-source Loopback0`|
|iBGP topology|R1, R2, R3 are clients of both R4 and R5|
|RR config|One peer-group on R4 and R5 carrying `remote-as`, `update-source`, `route-reflector-client`, and `next-hop-self`|
|eBGP peering|Physical interface addresses|
|OSPF network type|Point-to-point on the /30 links (see Design decisions)|

\---

## Config highlights

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Paste only the lines that matter, and add one sentence under each saying why. Full files go in `configs/`.

### Route reflector peer-group (R4 and R5)

```
(paste from your R4 config)
```

### iBGP client (R1 to R3)

```
(paste from your R1 config)
```

### eBGP peering (R4 to R6)

```
(paste from your R4 config)
```

### OSPF underlay

```
(paste from your R4 config)
```

\---

## Verification

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* For each check, paste the output or add a screenshot, then write two or three sentences: what you ran, what to notice, and what it means.

### 1\. OSPF neighbors and loopback reachability

```


R4# show ip ospf neighbor 

!\\\\\\\[R4 interface status](screenshots/01-show-ip-int-brief-R4.png)

R1# show ip ospf neighbor
R1# ping 10.255.0.4 source loopback 0
```

### 2\. iBGP sessions on the route reflectors

```
R4# show ip bgp summary
```

### 3\. Client view of the destination prefix (two paths)

```
R1# show ip bgp 10.200.8.0
```

### 4\. Next-hop reachability

```
R1# show ip route 10.200.8.0
R1# traceroute 10.200.8.1 source loopback 0
```

### 5\. Reflected route attributes

```
R2# show ip bgp 10.200.8.0
```

Look for the Originator and Cluster list lines.

### 6\. Redundancy test (shut R4, then R5)

```
R1# show ip bgp summary
R1# show ip bgp 10.200.8.0
```

### 7\. Local Preference steering

```
R1# show ip bgp 10.200.8.0
R1# traceroute 10.200.8.1 source loopback 0
```

\---

## Troubleshooting: bugs found and fixed

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Keep a Notepad log while you build and paste each problem here. This section is the most valuable part of the lab.

### Bug 1: (title)

**Symptom:
Root cause:
Fix:**

\---

## Design decisions

> \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*TODO:\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\* Answer each question in your own words, with evidence from the lab.

* **Where does `next-hop-self` go, and why on the route reflectors instead of the clients?**
* **Should R4 and R5 peer with each other, and as clients or non-clients?**
* **One cluster ID or two for the RRs?** What does each choice do to redundancy and to the number of duplicate paths the clients hold?
* **Why `ip ospf network point-to-point` on the /30 links?** On Ethernet, OSPF defaults to broadcast and elects a DR and BDR, which a two-router link does not need.
* **Which path did the clients pick when both upstream paths were equal, and which tiebreaker decided it?**

\---

## Files

|Folder|Contents|
|-|-|
|`configs/`|Final running configs, `R1.cfg` to `R8.cfg`|
|`verification/`|`show` command output as text files|
|`screenshots/`|Topology and annotated screenshots|
|`capture/`|Packet captures, if any|



