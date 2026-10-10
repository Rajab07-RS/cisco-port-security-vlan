# Network Segmentation & Layer 2 Port Security

## Overview
This project implements a hardened multi-VLAN corporate network architecture built in **Cisco Packet Tracer**. It demonstrates enterprise network segmentation using 802.1Q trunking, Inter-VLAN routing (Router-on-a-Stick), and physical switch-port hardening using **Sticky MAC Port Security** to prevent unauthorized access and rogue device connectivity.

---

## Network Architecture & Addressing Scheme

| Device | Interface | Zone / VLAN | IP Address / Subnet | Role |
| :--- | :--- | :--- | :--- | :--- |
| **PC-IT** | Fa0 | VLAN 10 (IT) | `192.168.10.10/24` | Authorized IT Workstation |
| **PC-HR** | Fa0 | VLAN 20 (HR) | `192.168.20.10/24` | HR Workstation |
| **SW-Floor1** | Fa0/1 | VLAN 10 | Access Port | Secured via Sticky MAC |
| **SW-Floor1** | Fa0/2 | VLAN 20 | Access Port | Standard Access Port |
| **SW-Floor1** | Gi0/1 | Trunk | N/A | 802.1Q Trunk to Router |
| **Core-Router** | Gi0/0.10 | VLAN 10 Gateway | `192.168.10.1/24` | Subinterface Gateway |
| **Core-Router** | Gi0/0.20 | VLAN 20 Gateway | `192.168.20.1/24` | Subinterface Gateway |

---

## Key Security Controls Implemented

### 1. Switch Port Security (Layer 2 Defense)
* **Interface `Fa0/1` Hardening:** Configured `switchport port-security` with `mac-address sticky` to dynamically learn and bind `PC-IT`'s hardware address into the running configuration.
* **Violation Policy:** Set violation mode to `shutdown`. Connecting an unapproved MAC address triggers an immediate port shutdown (`err-disabled` state), isolating the port until administrator intervention.
* **Host Limit:** Constrained maximum allowed MAC addresses to `1`.

### 2. Network Segmentation & Inter-VLAN Routing
* **VLAN Isolation:** Separated IT (`VLAN 10`) and HR (`VLAN 20`) broadcast domains on the Cisco 2960 switch.
* **Router-on-a-Stick:** Configured 802.1Q encapsulation subinterfaces on the Cisco 2911 router to control cross-VLAN traffic flow.

---

## Security Testing & Attack Simulation

### Test 1: Inter-VLAN Connectivity Verification
* **Action:** Issued `ping 192.168.20.10` from `PC-IT` (`192.168.10.10`).
* **Result:** **PASS** — Successful ICMP replies confirmed routing across subinterfaces `Gi0/0.10` and `Gi0/0.20`.

### Test 2: Rogue Device Attack Simulation
* **Action:** Disconnected `PC-IT` and attached an unauthorized device (`Attacker-Laptop`) with MAC address `0060.2F4A.1111` to port `Fa0/1`. Sent test ping to default gateway.
* **Result:** **PASS (Blocked)** — The switch instantly flagged a MAC mismatch, triggering a port violation.
  * Port `Fa0/1` transitioned to **`err-disabled`** state (link turned red).
  * `show port-security interface Fa0/1` confirmed `SecurityViolation Count: 1`.

---

## Incident Response & Remediation Procedure
To recover a port placed in an `err-disabled` state following a security violation:
1. Identify and disconnect the rogue MAC address.
2. Re-attach the authorized workstation.
3. Access switch CLI and reset the interface:
   ```text
   SW-Floor1(config)# interface FastEthernet0/1
   SW-Floor1(config-if)# shutdown
   SW-Floor1(config-if)# no shutdown
