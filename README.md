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

### 1.3 Advanced IOMMU & GPU Passthrough (VFIO)

**The Hardware Acceleration Requirement:**
To support the heavy rendering workflows of the Quasi-VDI endpoints, the Proxmox virtual machines required direct access to physical GPU compute. 

**vGPU Splitting vs. Discrete Passthrough:**
Initial architectural research involved evaluating GPU resource splitting (vGPU). While NVIDIA artificially restricts vGPU capabilities on consumer-grade GTX/RTX cards, I explored community-developed `vgpu_unlock` kernel hooks that bypass this restriction on GTX architectures. However, to guarantee zero-latency performance and maximize raw compute for the CAD/gaming workloads, I opted for **Discrete PCIe Passthrough**—dedicating one full physical GPU to each high-performance virtual machine.

**The Headless Host Challenge:**
Each Proxmox host was equipped with 2 GPUs, and I needed to pass both through to the VMs. By default, the Debian-based Proxmox host OS will capture the primary GPU in the primary PCIe slot to output the console display (the framebuffer). If the host OS holds the GPU, it cannot be passed to a VM. 

**Implementation (Framebuffer Isolation):**
I engineered the host to operate strictly headlessly, managed exclusively over the network. To achieve this, I modified the GRUB bootloader to prevent the host kernel from initializing a display output, effectively isolating the GPUs so they could be bound to the `vfio-pci` driver.

**Configuration Steps:**
Modified `/etc/default/grub` to include IOMMU enablement and framebuffer blacklisting:

```bash
# Enabled IOMMU and disabled the host video framebuffers
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt initcall_blacklist=sysfb_init video=simplefb:off video=vesafb:off video=efifb:off"

# Update GRUB to apply boot parameters
update-grub

# Bind the specific NVIDIA GPU and Audio hardware IDs to the VFIO driver
echo "options vfio-pci ids=10de:xxxx,10de:xxxx disable_vga=1" > /etc/modprobe.d/vfio.conf

# Update initramfs to ensure VFIO captures the GPUs before the Nouveau/NVIDIA drivers
update-initramfs -u -k all



### 4.5 OS Optimization & Telemetry Hardening

**The Resource Overhead Challenge:**
Standard Windows 11 deployments include significant consumer-level bloatware, background telemetry, and scheduled tasks. In a virtualized environment, this inherent OS overhead severely degrades VM density and consumes critical CPU/RAM resources that must be preserved for the CAD and gaming workloads. 

**Implementation (Base Image & Scripting):**
To maximize resource efficiency and guarantee that 99% of compute power was dedicated to the primary applications, I engineered a highly optimized, stripped-down Windows environment for the virtual machines.

* **Lightweight Base OS:** Deployed **Tiny11** as the baseline operating system. This drastically reduced the default storage footprint and idle RAM consumption by bypassing standard Windows 11 hardware checks and stripping out native bloatware prior to installation.
* **Automated Debloating:** Utilized the **Chris Titus Tech (CTT) Windows Utility** and **Winhance** scripts post-installation to systematically remove residual consumer applications, Cortana, and unnecessary default AppX packages.
* **Telemetry Hardening & Privacy:** Standard Windows telemetry acts as embedded spyware, constantly phoning home and eating network/CPU bandwidth. To mitigate this, I executed targeted **Registry (Regedit)** modifications to disable background data collection. Finally, applied **O&O ShutUp10++** to enforce strict system-level privacy policies, disabling cloud syncs and preventing unwanted background updates from disrupting rendering sessions.

**The Impact:**
By meticulously hardening the OS, idle CPU utilization was reduced to near-zero, and baseline RAM consumption was cut by over 50%. This optimization was critical in allowing the 2 centralized Proxmox hosts to comfortably serve 9 high-performance VMs simultaneously without bottlenecking.


## 💾 3. Storage Architecture & Network Integration

### 3.1 The Physical I/O Limitation
**The Challenge:**
In a fully virtualized "Quasi-VDI" architecture, end-users interface with the system via thin clients. Because the actual compute and operating systems live on remote Proxmox host servers, traditional physical I/O methods (such as inserting a USB flash drive to transfer files) are impossible. A robust, centralized storage solution was required to allow users to move large CAD and rendering files in and out of their virtual machines.

### 3.2 TrueNAS Deployment & Centralized Storage
**The Solution:**
To create a high-performance data backbone, I deployed a dedicated **TrueNAS** virtual appliance within the Proxmox environment. This isolated the storage management from the compute nodes and provided a scalable platform for enterprise file sharing.

**Implementation & Configuration:**
* **ZFS Storage Pool:** Configured the underlying storage utilizing the ZFS file system, ensuring high data integrity, protection against bit-rot, and efficient read/write caching for large rendering files.
* **SMB (Server Message Block) Provisioning:** Engineered network shares utilizing the SMB protocol. This protocol was specifically selected for its native integration with the optimized Windows 11 VMs.
* **Network Drive Mapping:** The SMB shares were broadcasted across the secure LAN and automatically mapped as Network Drives within the guest VMs. 

**The Impact:**
This architecture completely bypassed the physical I/O limitations of the thin clients. Users can now seamlessly drag and drop massive project files across the network directly into their rendering VMs at full Gigabit LAN speeds, creating a frictionless, centralized storage ecosystem.
