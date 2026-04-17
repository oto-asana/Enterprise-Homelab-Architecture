# Enterprise Infrastructure & Virtualization Simulation

## 📌 Project Overview
An intensive enterprise infrastructure simulation designed to demonstrate advanced networking, systems administration, and virtualization capabilities. Architected and deployed entirely from scratch, this lab serves as a practical showcase of expertise in deploying and managing high-performance Proxmox VE environments, TrueNAS storage architectures, and secure Layer 2 network segmentation.

**Core Technologies:** Proxmox VE, TrueNAS, Debian, VLANs, Sunshine VDI, NVR Systems.

### Architectural Note
> Within the network topology, configuration warnings may be noted on the 1GB switch nodes. This is an intentional design choice to accurately simulate constraints utilizing unmanaged switches operating within a mixed-hardware environment, reflecting realistic challenges often found in mid-level enterprise physical infrastructures.

---

🖥️ High-Performance "Quasi-VDI" Architecture
The Challenge:
Deploying a fully open-source Virtual Desktop Infrastructure (VDI) that remains stable under heavy graphical workloads (such as gaming and CAD rendering) is a significant industry challenge. Traditional free connection brokers often introduce severe latency, drop connections, or lack robust GPU-passthrough support.

The Solution:
To bypass the limitations of standard free VDI software, I architected a high-performance "Quasi-VDI" environment utilizing Proxmox VE. This custom architecture prioritizes raw stability, graphics performance, and resource efficiency over centralized login convenience.

Hardware Topology:

Host Nodes: 2 high-compute Proxmox servers equipped with 4 high-performance GPUs.

Endpoints (Thin Clients): 9 Ultra-Small Form Factor (USFF) desktop PCs with low baseline specifications.

Architectural Workflow:
Instead of relying on a centralized VDI broker, the 9 thin clients leverage a high-throughput LAN topology to stream hardware-accelerated workloads directly from the virtual machines. Authentication is handled directly at the host-OS level rather than through a unified web portal. While this shifts the login process to the VM itself, it completely eliminates broker latency and overhead, resulting in a drastically smoother rendering experience for the end-user.
