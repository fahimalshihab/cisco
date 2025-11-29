
# cisco

Great — I turned your whole lab text into a **clear, practical self-training program** so you can *get trained properly* using exactly that content. No fluff — just step-by-step lessons, hands-on labs, verification commands, quizzes, and troubleshooting checks. Follow it and you’ll master the whole lab.

---

# Fast Training Plan — Practice Using Your Lab (6 modules)

Each module = short theory → hands-on steps → verification → mini quiz. Do one module per day (or faster if you prefer).

---

## Module 1 — VLANs & VTP (Goal: create VLANs and propagate them)

* **Theory (5 min):** What is VLAN and VTP (you already have short defs).
* **Hands-on**

  1. On **DLSW**: set hostname, configure VTP server and domain/password.

     ```text
     conf t
     hostname DLSW
     vtp mode server
     vtp domain xyz
     vtp password ccna
     ```
  2. Create VLANs on DLSW:

     ```text
     vlan 2 name sales
     vlan 3 name marketing
     vlan 4 name administration
     vlan 5 name it
     vlan 6 name wireless
     ```
  3. On **ALSW1/ALSW2**: set VTP client and same domain/password.

     ```text
     conf t
     hostname ALSW1
     vtp mode client
     vtp domain xyz
     vtp password ccna
     ```
* **Verify**

  ```text
  show vtp status
  show vlan brief       (on DLSW and ALSW1/ALSW2)
  ```
* **Mini quiz (2 Qs)**

  * Why use VTP?
  * What happens if VTP domain mismatches?

---

## Module 2 — Trunks, EtherChannel basics & switchport ports

* **Theory (5 min):** trunk vs access ports; purpose of EtherChannel (PAgP vs LACP).
* **Hands-on**

  1. Configure trunk ports (example ranges):

     ```text
     DLSW#conf t
     int range g1/0/2-5
     switchport mode trunk
     ```
  2. Configure EtherChannel Group 1 (PAgP) between DLSW ↔ ALSW1:

     ```text
     DLSW(config)#int range g1/0/2-3
     channel-group 1 mode desirable
     interface port-channel 1
     switchport mode trunk
     ```

     On ALSW1 use same but `channel-group 1 mode desirable`.
  3. EtherChannel Group 2 (LACP) DLSW ↔ ALSW2:

     ```text
     DLSW(config)#int range g1/0/4-5
     channel-group 2 mode active
     interface port-channel 2
     switchport mode trunk
     ```
* **Verify**

  ```text
  show etherchannel summary
  show interfaces trunk
  show interfaces port-channel brief
  ```
* **Mini quiz**

  * PAgP vs LACP: which is Cisco proprietary?
  * How to check active members of port-channel?

---

## Module 3 — Inter-VLAN Routing (SVIs) & IP plan

* **Theory (5 min):** SVI purpose and ip routing on multilayer switch.
* **Hands-on**

  1. Enable routing on DLSW:

     ```text
     conf t
     ip routing
     ipv6 unicast-routing
     ```
  2. Create SVIs and assign IPv4/IPv6 gateway addresses (use your values):

     ```text
     int vlan 1
       ip address 172.16.1.1 255.255.255.0
       ipv6 address 2001:211:A:1::1/64
       no shutdown
     int vlan 12
       ip address 172.16.2.1 255.255.255.0
       ipv6 address 2001:211:A:2::1/64
     ...
     ```
* **Verify**

  ```text
  show ip route
  show ip interface brief
  show ipv6 interface brief
  ```
* **Mini quiz**

  * Why `ip routing` on switch?
  * What is SVI vs physical routed port?

---

## Module 4 — DHCPv4 & DHCPv6 (stateful) on DLSW

* **Theory (5 min):** DHCP pools, excluded addresses, DHCPv6 stateful.
* **Hands-on**

  1. Exclude first 10 IPs for Sales & Marketing:

     ```text
     ip dhcp excluded-address 172.16.2.1 172.16.2.10
     ip dhcp excluded-address 172.16.3.1 172.16.3.10
     ```
  2. Create DHCP pools:

     ```text
     ip dhcp pool sales
       network 172.16.2.0 255.255.255.0
       default-router 172.16.2.1
       dns-server 8.8.8.8
     ip dhcp pool marketing
       network 172.16.3.0 255.255.255.192
       default-router 172.16.3.1
       dns-server 8.8.8.8
     ```
  3. DHCPv6 pool + attach to SVI:

     ```text
     ipv6 dhcp pool sales-v6
       address prefix 2001:211:A:2::/64
       dns-server 2001:4860:4860::8888
     int vlan 12
       ipv6 dhcp server sales-v6
       ipv6 nd managed-config-flag
     ```
* **Verify**

  * On a PC in Sales: confirm IPv4 & IPv6 assigned.
  * Commands:

    ```text
    show ip dhcp binding
    show ipv6 dhcp binding
    ```
* **Mini quiz**

  * Why exclude addresses?
  * Difference between DHCPv6 stateless and stateful?

---

## Module 5 — STP, PortFast, BPDU Guard & Port Security

* **Theory (5 min):** Rapid-PVST basics, PortFast for access ports, BPDU guard to prevent loops, port-security basics.
* **Hands-on**

  1. Enable Rapid-PVST:

     ```text
     spanning-tree mode rapid-pvst
     ```
  2. PortFast + BPDU guard on ALSW1 access ports:

     ```text
     int range f0/1-2
       spanning-tree portfast
       spanning-tree bpduguard enable
     ```
  3. Port security on ALSW2:

     ```text
     int range f0/1-2
       switchport port-security
       switchport port-security maximum 1
       switchport port-security mac-address sticky
       switchport port-security violation shutdown
     ```
* **Verify**

  ```text
  show spanning-tree summary
  show port-security
  show port-security interface f0/1
  ```
* **Mini quiz**

  * What happens when BPDU guard receives a BPDU on a PortFast port?
  * Sticky MAC vs static MAC?

---

## Module 6 — Testing, Troubleshooting & Validation checklist

* **Hands-on tests**

  1. From a Sales PC: `ping 172.16.2.1` (gateway), `ping 172.16.3.x` (other VLAN), `ping 8.8.8.8` (if routing to router exists).
  2. `show vlan brief` on ALSW1/ALSW2 to confirm VLANs present.
  3. `show etherchannel summary` to confirm LACP/PAgP up.
  4. `show ip dhcp binding` confirm leases.
* **Common troubleshooting commands**

  ```text
  show ip interface brief
  show interfaces trunk
  show running-config
  debug ip dhcp server events   (use carefully)
  show logging
  ```
* **Validation checklist**

  * VLANs present on all switches
  * Trunk and port-channels up
  * SVIs up with IPs
  * DHCP leases assigned to Sales/Marketing
  * PCs can ping gateways and inter-VLAN hosts

---

# Extra practice exercises (do after modules)

1. Break one trunk (shutdown an uplink) and confirm EtherChannel failover.
2. Add a wrong VTP password on ALSW1 and observe VLAN absent — fix it.
3. Configure a static host on a port violating port-security and see violation action.
4. Remove DHCP exclusion and attempt a conflicting static IP to see behavior.

---

# Quick reference cheat-sheet (copy this)

* Show VLANs: `show vlan brief`
* Show VTP: `show vtp status`
* Show trunks: `show interfaces trunk`
* Show port-channels: `show etherchannel summary`
* Show SVIs: `show ip interface brief` / `show ipv6 interface brief`
* DHCP bindings: `show ip dhcp binding` / `show ipv6 dhcp binding`
* Enable routing: `ip routing` / `ipv6 unicast-routing`

---

# Assessment (pass = complete)

* Create VLANs on DLSW and confirm on ALSW1/ALSW2 via VTP.
* Configure SVIs + `ip routing` and be able to ping across VLANs from PCs.
* Sales and Marketing PCs obtain IPv4 and IPv6 (stateful) addresses.
* EtherChannel groups up and carry VLANs.
* STP PortFast + BPDU guard and port-security work as expected.

---

If you want, I can:

* Convert this into a **2-week calendar** with daily checkboxes.
* Generate **exact commands** for Tasks 2–6 in the same compact style.
* Produce a **PDF** of the training plan and cheat sheet.

Which one next?
