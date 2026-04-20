# 🖧 Hardware Appliance Configuration Guidelines

Unlike Debian-based hypervisors or SDN controllers, physical appliances such as NVRs and PBX systems utilize proprietary Web GUIs. The following guidelines define the architectural and security baselines required for deploying any telecommunications or surveillance appliance within this infrastructure.

---

## 1. Surveillance Infrastructure (Camera Server / NVR)

Regardless of the hardware vendor (e.g., Dahua, Hikvision, Ubiquiti), the NVR must be provisioned according to strict IoT air-gapping principles.

### Security & Isolation Baselines
* **Management Interface Isolation:** The NVR's primary uplink (WAN interface) must be strictly assigned to the Management subnet (**VLAN 99**). Access to the web login portal is explicitly denied to the Production (VLAN 10) and Guest (VLAN 20) networks.
* **IoT Air-Gapping:** IP Cameras must never be patched directly into the main corporate LAN. They must be physically connected to the NVR's downstream POE ports.
* **Internal NAT Routing:** The NVR must be configured to act as a DHCP server for its downstream cameras, utilizing a completely isolated, non-routable subnet (e.g., `10.0.0.0/24`). This ensures cameras cannot reach the internet and cannot be accessed laterally by a compromised user endpoint.

---

## 2. Unified Communications (VoIP / PBX Server)

Voice traffic is highly sensitive to network congestion. To ensure clear communication and prevent infrastructure compromise, the PBX (e.g., CooVox) must adhere to both security and Quality of Service (QoS) baselines.

### Quality of Service (QoS) & Hardware Separation
* **Bandwidth Contention Mitigation:** Voice packets are highly sensitive to jitter and latency. To prevent dropped packets when the data network is under heavy load (e.g., during large CAD file transfers), voice traffic must be logically or physically separated. 
* **Deployment Strategy:** The PBX and IP phones should ideally be placed on a dedicated **Voice VLAN**. In environments where network traffic is heavily saturated and managed switches lack deep QoS capabilities, deploying a physical "air-gap" (a completely separate unmanaged switch dedicated solely to IP phones) is an acceptable strategy to guarantee zero bandwidth contention.

### Security & SIP Provisioning
* **Portal Security:** Similar to the NVR, the PBX administrative Web GUI must reside exclusively on **VLAN 99** to prevent unauthorized users from intercepting voice traffic or tampering with extension routing.
* **SIP Configuration Baseline:** * Disable auto-provisioning across the open network to prevent rogue devices from registering.
  * Statically create SIP accounts and manually assign SIP numbers/extensions to specific IP phone MAC addresses or static IPs. 
  * Utilize strong, randomized alphanumeric passwords for individual SIP registrations, rather than default PINs.
