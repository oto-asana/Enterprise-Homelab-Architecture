# Enterprise Infrastructure & Virtualization Simulation

## 📌 Project Overview
An intensive enterprise infrastructure simulation designed to demonstrate advanced networking, systems administration, and virtualization capabilities. Architected and deployed entirely from scratch, this lab serves as a practical showcase of expertise in deploying and managing high-performance Proxmox VE environments, TrueNAS storage architectures, and secure Layer 2 network segmentation.

**Core Technologies:** Proxmox VE, TrueNAS, Debian, VLANs, Sunshine VDI, NVR Systems.

### Architectural Note
> Within the network topology, configuration warnings may be noted on the 1GB switch nodes. This is an intentional design choice to accurately simulate constraints utilizing unmanaged switches operating within a mixed-hardware environment, reflecting realistic challenges often found in mid-level enterprise physical infrastructures.

---

## 🖥️ 4. High-Performance "Quasi-VDI" Architecture

### 4.1 The Engineering Challenge
Deploying a fully open-source Virtual Desktop Infrastructure (VDI) that remains stable under heavy graphical workloads (such as gaming and CAD rendering) is a significant industry challenge. Standard connection brokers often introduce severe latency, drop connections, or lack robust GPU-passthrough support for demanding Windows environments.

To solve this, I architected a custom "Quasi-VDI" environment utilizing Proxmox VE. This architecture prioritizes raw stability, graphics performance, and resource efficiency over centralized login convenience by utilizing direct host-OS authentication.

**Hardware Topology:**
* **Host Nodes:** 2 high-compute Proxmox servers equipped with 4 high-performance GPUs.
* **Endpoints (Thin Clients):** 9 Ultra-Small Form Factor (USFF) desktop PCs with low baseline specifications.

### 4.2 Protocol Selection & Troubleshooting Analysis
To connect the 9 thin clients to the resource pools, multiple streaming protocols were heavily tested and evaluated against our high-performance requirement:

* **Iteration 1: SPICE Protocol (via PVE-VDIClient)**
  * *Pros:* Excellent native Proxmox integration and seamless VM authentication. Very stable on Linux guests.
  * *Cons:* Highly unstable when handling demanding, GPU-accelerated processes on Windows VMs. The protocol could not maintain a playable/renderable framerate under heavy load.
* **Iteration 2: Microsoft RDP (Remote Desktop Protocol)**
  * *Pros:* Vastly improved stability over SPICE. Connection drops and freezing were eliminated.
  * *Cons:* Windows 11 enforces a hardcoded 30 FPS lock on native RDP connections. Despite registry modifications and group policy edits, the WDDM graphics driver limitations rendered it unacceptable for CAD and gaming workloads.
* **Final Implementation: Sunshine & Moonlight Streaming**
  * *Result:* Deployed the open-source **Sunshine** streaming host on the Proxmox VMs, paired with **Moonlight** clients on the 9 USFF endpoints. 
  * *Impact:* Successfully bypassed the Windows 11 30 FPS lock. This protocol leverages direct hardware encoding/decoding, delivering ultra-low latency, 60+ FPS streaming perfectly suited for heavy rendering and gaming, while seamlessly pooling the resources of the 2 master servers.
