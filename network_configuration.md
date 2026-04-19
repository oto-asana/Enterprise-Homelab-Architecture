Here is the raw configuration data cleaned up, standardized, and formatted professionally. I have applied standard Linux network formatting, added comments to explain *why* the settings are there, and fixed a few minor IP/tagging typos from your raw text to ensure it perfectly matches the architecture described in your README.

You can save this directly into your GitHub repository inside a `configs/` folder as a file named `network_configuration.md`.

***

```markdown
# 🖧 Core Network & SDN Configurations

This document contains the raw network configuration files for the Proxmox hypervisor nodes, the Datacenter Manager (SDN), and the physical switch topology. 

---

## 1. Proxmox VE Nodes (VLAN-Aware Bridges)
To allow virtual machines to be assigned to different VLANs (10, 20, 99), the core Proxmox network bridges must be configured as `vlan-aware`. 

### Host 01 Configuration
**File:** `/etc/network/interfaces`
```text
auto lo
iface lo inet loopback

# Physical Interface (Replace 'nic0' with actual interface name, e.g., eno1 or eth0)
auto nic0
iface nic0 inet manual

# VLAN-Aware Linux Bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.99.11/16
    gateway 192.168.99.1
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

### Host 02 Configuration
**File:** `/etc/network/interfaces`
```text
auto lo
iface lo inet loopback

# Physical Interface
auto nic0
iface nic0 inet manual

# VLAN-Aware Linux Bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.99.12/16   # Incremented IP for Host 2
    gateway 192.168.99.1
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

---

## 2. Datacenter Manager (Software-Defined DHCP)
Because the physical switch is only L2+, a centralized `dnsmasq` service was deployed on the Datacenter Manager to handle DHCP provisioning for the Management VLAN (and future subnets).

### Installation
```bash
sudo apt update && sudo apt install dnsmasq -y
```

### Network Interface Configuration
**File:** `/etc/network/interfaces`
```text
auto lo
iface lo inet loopback

auto nic0
iface nic0 inet static
    address 192.168.99.3/16    # Static IP assigned to the DM on the management subnet
    gateway 192.168.99.1
```

### DNSMasq DHCP Configuration
**File:** `/etc/dnsmasq.conf`
```text
# --- General Settings ---
domain-needed
bogus-priv

# Listen on specific VLAN sub-interfaces
interface=eth0.10
interface=eth0.20
interface=eth0.99

# --- VLAN 99 (Infrastructure Management) ---
# Distributes IPs from .50 to .150 on the 192.168.0.0/16 subnet
dhcp-range=set:vlan99,192.168.99.50,192.168.99.150,255.255.0.0,24h
dhcp-option=tag:vlan99,option:router,192.168.99.1
dhcp-option=tag:vlan99,option:dns-server,192.168.99.1
```

---

## 3. Physical Switch Port Topology Rules
To ensure the 802.1Q tags successfully pass from the Proxmox host into the physical network, the switch ports must be configured with specific tagging rules:

* **Proxmox Host Uplink Ports:** Must be configured as **Trunk Ports** (Tagged). This allows the switch to read and transport traffic from all VMs, regardless of which VLAN (10, 20, 99) they are assigned to in the Proxmox GUI.
* **Datacenter Manager (SDN) Port:** * *Current Setup:* Can be configured as an **Access Port** (Untagged) bound to VLAN 99, because it currently only distributes IPs to the management network.
  * *Future Scaling:* If `dnsmasq` is expanded to serve DHCP to VLAN 10 and VLAN 20, this port must be converted to a **Trunk Port** so the DHCP broadcast packets can traverse multiple subnets.
```

***

### What I fixed/improved for your portfolio:
1.  **Host 2 IP Fix:** Your raw text had Host 1 and Host 2 both set to `192.168.99.11`. I changed Host 2 to `.12` to prevent an IP conflict and match your README.
2.  **DNSMasq Tags:** Your raw text used `vlan10` tags for the `192.168.99.x` subnet. I corrected the tags to `vlan99` so that the code logically matches the subnet it's serving.
3.  **Datacenter Manager IP:** Your raw text had the DM interface set to `.1` (which is the gateway). I corrected it to `.3` to match the topology you described in the previous section.
4.  **Loopback Formatting:** Added the proper `iface lo inet loopback` syntax which is standard for Debian/Proxmox systems. 

This looks incredibly clean and proves you know how to write standard configuration-as-code documentation!
