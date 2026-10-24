# hsrp-redundant-default-gateway
First Hop Redundancy Protocol (FHRP) architecture utilizing HSRP Group 10 across Layer 3 distribution/core switches.

# Lab 6: HSRP Redundancy & High Availability Architecture

A complete `README.md` based on your **HSRP Layout** topology and network specifications.

---

## 🌐 Quick Overview & Network Specs

* **🟢 Architecture & Topology:** First Hop Redundancy Protocol (FHRP) architecture utilizing **HSRP Group 10** across Layer 3 distribution/core switches (`L3-SW-1` and `L3-SW-2`) providing dual-homed internet gateway redundancy to downstream Access switches (`Dist1`, `Access 1-1`, `Access 1-2`) and end-user hosts.
* **🔵 Layer 3 Switching & Core Gateways:**
* **L3-SW-1 (Active Router):** Primary default gateway interface assigned physical IP `192.168.10.1/24`, configured with higher HSRP priority (e.g., `110`) and preemption enabled.
* **L3-SW-2 (Standby Router):** Secondary default gateway assigned physical IP `192.168.10.2/24`, serving as a hot standby with default priority (`100`).
* **Virtual IP (VIP):** `192.168.10.3` (HSRP Group 10) shared virtually between both Layer 3 switches to act as the default gateway for all end hosts.


* **🟣 WAN / ISP Uplinks:**
* Point-to-Point WAN link between `L3-SW-1` and ISP Router `2911`: `203.0.113.0/30` (`.1` on switch side).
* Point-to-Point WAN link between `L3-SW-2` and ISP Router `2911`: `203.0.113.4/30` (`.2` on switch side).
* External ISP Loopback / Server Destination: `8.8.8.8`.


* **🔴 Downstream Host Allocations (`192.168.10.0/24` LAN Domain):**
* **PC4:** `192.168.10.4 /24` (Default Gateway: `192.168.10.3`)
* **PC5:** `192.168.10.5 /24` (Default Gateway: `192.168.10.3`)
* **PC6:** `192.168.10.6 /24` (Default Gateway: `192.168.10.3`)
* **PC7:** `192.168.10.7 /24` (Default Gateway: `192.168.10.3`)



---

## 💡 Key Technical Insights & What This Lab Demonstrates

* 🔍 **First Hop Redundancy Protocol (FHRP) Mastery:** Demonstrates how Cisco's proprietary Hot Standby Router Protocol (HSRPv1/v2) prevents single points of failure (SPOF) at the default gateway layer by masking physical interfaces behind a single virtual IP (`192.168.10.3`) and virtual MAC address (`0000.0c07.ac0a`).
* ⚡ **Seamless Failover & Active/Standby State Machine:** Audits HSRP state transitions (`Initial` ➔ `Listen` ➔ `Speak` ➔ `Standby` ➔ `Active`). Proves sub-second transparent failover where end-user sessions (ping/HTTP traffic to `8.8.8.8`) continue uninterrupted if `L3-SW-1` or its uplink fails.
* 🛠️ **Interface Tracking & Preemption:** Configures uplink interface tracking (`track <interface-id>`) on `L3-SW-1` so that if its WAN connection to `203.0.113.0/30` fails, its priority dynamically decrements, allowing `L3-SW-2` to gracefully preempt and assume the Active state.
* 🛡️ **Edge High-Availability & Routing Hygiene:** Highlights real-world enterprise network design principles by configuring all client workstations (`PC4`-`PC7`) with the Virtual IP (`192.168.10.3`) as their gateway, guaranteeing internet reachability regardless of single hardware or line outage events.

---

## 🛠️ Essential Cisco CLI Configuration Snippets

### **L3-SW-1 (Active Gateway Configuration)**

```cisco
interface GigabitEthernet0/1
 description LAN Gateway Interface
 ip address 192.168.10.1 255.255.255.0
 standby 10 ip 192.168.10.3
 standby 10 priority 110
 standby 10 preempt
 standby 10 track GigabitEthernet0/2 15
!
interface GigabitEthernet0/2
 description Uplink to ISP Router
 ip address 203.0.113.1 255.255.255.252

```

### **L3-SW-2 (Standby Gateway Configuration)**

```cisco
interface GigabitEthernet0/1
 description LAN Gateway Interface
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.3
 standby 10 priority 100
 standby 10 preempt
!
interface GigabitEthernet0/2
 description Backup Uplink to ISP Router
 ip address 203.0.113.2 255.255.255.252

```

---

## 🧪 Verification & Audit Commands

```bash
# Verify HSRP status, active/standby roles, priority, and virtual IP
show standby brief
show standby

# Verify Layer 3 ARP mapping for the HSRP Virtual IP
show ip arp 192.168.10.3

# Test failover reachability from client PC (continuous ping)
ping 8.8.8.8 -t

```
