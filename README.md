# Enterprise Infrastructure & Virtualization Simulation

## 📌 Project Overview
An intensive enterprise infrastructure simulation designed to demonstrate advanced networking, systems administration, and virtualization capabilities. Architected and deployed entirely from scratch, this lab serves as a practical showcase of expertise in deploying and managing high-performance Proxmox VE environments, TrueNAS storage architectures, and secure Layer 2 network segmentation.

**Core Technologies:** Proxmox VE, TrueNAS, Debian, VLANs, Sunshine VDI, NVR Systems.

### Architectural Note
> Within the network topology, configuration warnings may be noted on the 1GB switch nodes. This is an intentional design choice to accurately simulate constraints utilizing unmanaged switches operating within a mixed-hardware environment, reflecting realistic challenges often found in mid-level enterprise physical infrastructures.

---

## 🏗️ 1. Hypervisor Environment (Proxmox VE)
### 1.1 Core Setup
*(Insert your baseline Proxmox installation and network bridge configuration here)*

### 1.2 Proxmox Troubleshooting & Maintenance
**Issue:** VM "Ghosting" – Deleted virtual machines remain visible in the GUI, or newly created VMs with different IDs fail to populate.
**Root Cause:** The `pveproxy` service (web interface) desyncs from the backend configuration files stored in `/etc/pve/qemu-server/`.

**Resolution Protocol:**
1. **Service Restart:** Force the API and status services to reload without interrupting running VMs.
   ```bash
   systemctl restart pveproxy
   systemctl restart pvestatd
