# VirtualBox Networking Modes — A Practical Comparison

> Supporting document for
> [NETWORKWALKS-B082-WK1-PM1-CYBERSECURITY-LAB-SETUP](../README.md).
> Written to justify the choice of **NAT Network** for this lab.

VirtualBox offers five ways to attach a virtual network adapter. They are not
interchangeable, and the choice determines what a lab can and cannot do. Below is
what each one actually does, and why four of them were rejected.

---

## The five modes at a glance

| Capability | NAT | NAT Network | Bridged | Host-Only | Internal |
|---|:--:|:--:|:--:|:--:|:--:|
| VM → Internet | ✅ | ✅ | ✅ | ❌ | ❌ |
| VM → VM | ❌ | ✅ | ✅ | ✅ | ✅ |
| VM → Host | ✅ | ✅ | ✅ | ✅ | ❌ |
| Host → VM | ⚠️ port forwarding | ⚠️ port forwarding | ✅ | ✅ | ❌ |
| VM visible on physical LAN | ❌ | ❌ | ✅ | ❌ | ❌ |
| Needs a DHCP server | built-in | built-in | your router | built-in | ❌ manual |
| Typical use | Quick internet for one VM | **Multi-VM security lab** | VM acting as a real network host | Isolated host↔VM management | Fully air-gapped VM group |

---

## 1. NAT

Each VM gets its own private, invisible NAT engine. The VM sits behind it on
`10.0.2.0/24` and reaches the internet through the host.

**Strength.** Zero configuration. Works the moment the VM boots.

**The fatal limitation for a lab:** every VM gets a *separate* NAT engine. Two VMs
both using plain NAT each believe they are `10.0.2.15` and have no path to each
other. They are not on a shared network — they are in isolated tunnels.

**Rejected because** Week 2 onwards requires an attacker VM and a target VM that can
see each other.

---

## 2. NAT Network ✅ *chosen*

A NAT engine shared by every VM attached to the same named network. VirtualBox
creates a virtual switch; all attached VMs sit on one subnet, and one NAT gateway
carries their traffic out to the host's network.

**Configuration used in this lab:**

| Setting | Value |
|---|---|
| Name | `NatNetwork` |
| IPv4 prefix | `10.0.0.0/24` |
| DHCP | Enabled |
| IPv6 | Disabled |
| Gateway | `10.0.0.1` |
| Usable host range | `10.0.0.2` – `10.0.0.254` |

**Why this mode wins.** It is the only one that satisfies all three lab requirements
simultaneously:

1. **Internet access** — needed for `apt update` and downloading tools.
2. **VM-to-VM reachability** — needed for scanning and exploitation exercises.
3. **Isolation from the physical LAN** — needed so that scanning tools never touch
   my home network or my ISP's.

**Trade-off.** The host cannot initiate connections into a VM without a port
forwarding rule (Network Manager → NAT Networks → Port Forwarding). For this lab
that is not a limitation — it is a feature, since it means nothing on the host can
accidentally reach the lab either.

---

## 3. Bridged Adapter

The VM's virtual NIC is bridged onto the host's physical NIC. The VM gets an address
from the real router and appears on the physical LAN as an independent device.

**Strength.** The VM is a genuine network peer. Other devices can reach it directly.

**Rejected because — and this is the important one — a Kali box on the home LAN is a
liability.** An `nmap` sweep would hit the router, family devices, printers, and any
neighbouring device on the same segment. On a shared, university, or corporate
network, running scanning tools from a bridged VM can violate acceptable-use policy
and may be indistinguishable from an attack. Bridged mode is appropriate when the
VM is meant to *be* a network host, not when it is meant to be a weapons range.

---

## 4. Host-Only Adapter

Creates a virtual network between the host and the VMs. No internet access at all.

**Strength.** Complete outbound isolation with full host↔VM reachability — ideal for
detonating malware samples or for a management interface.

**Rejected as the sole adapter** because the lab needs to update packages and fetch
tools. It remains useful as a **second adapter** alongside NAT Network, which is a
common production lab pattern:

- Adapter 1 → NAT Network (internet + VM-to-VM)
- Adapter 2 → Host-Only (guaranteed host access even if the lab network is broken)

This is a likely extension for a later week.

---

## 5. Internal Network

A virtual switch connecting VMs to each other and to nothing else — not the host,
not the internet.

**Strength.** True air gap. The correct choice for detonating live malware.

**Rejected because** with no host access and no internet, it is too restrictive for
a general-purpose lab. Worth revisiting for a dedicated malware analysis segment.

---

## Decision summary

| Requirement | Mode that satisfies it |
|---|---|
| Internet for updates and tools | NAT, NAT Network, Bridged |
| Attacker and target VMs must see each other | NAT Network, Bridged, Host-Only, Internal |
| Must not touch the physical home LAN | NAT, NAT Network, Host-Only, Internal |
| **Intersection of all three** | **NAT Network** ✅ |

---

## A note on promiscuous mode

Within the chosen mode, Adapter 1 is set to **Promiscuous Mode: Allow All**.

By default a network interface discards Ethernet frames whose destination MAC is not
its own. Promiscuous mode disables that filter, so the interface passes *every* frame
on the segment up to the operating system. This is what allows Wireshark and
`tcpdump` to observe traffic between other machines on the same virtual switch —
essential for the ARP spoofing and traffic analysis exercises that follow in later
weeks.

In production this setting would be a red flag: it is precisely what an attacker
enables after compromising a host in order to harvest credentials from the wire.
Here it is a deliberate, documented choice appropriate to an isolated lab.

---

## Reference

- [VirtualBox Manual — Chapter 6: Virtual Networking](https://www.virtualbox.org/manual/ch06.html)
