---
icon: mdi mdi-cellphone-text
order: 10
layout: page
---

Below is a detailed overview of all the hardware currently used in my homelab, from servers to printers.

The main devices in my homelab are

- Custom Built [Unraid](/tags/unraid) Powered Server (NAS)
- 3x Dell Optiplex 3060's as a [Proxmox](/tags/proxmox) Cluster
- 8 Port POE 2.5gbe Switch with x1 10gbe SFP+

The "NAS" system has the following specification
## NAS Server - Custom Build
![nas.png](/assets/img/old/nas.png)
-   **CPU:** AMD Ryzen 5 5500GT
-   **Motherboard:** ASUS ROG STRIX X570-F GAMING
-   **RAM:** 64GB 3200mt/s (4x16GB) DDR4
-   **M.2 Devices**
    -   Samsung PM891 256GB NVMe SSD (Cache)
    -   Samsung SM961 512GB NVMe SSD (AI Cache)
-   **SATA HDDs**
    -   Seagate BarraCuda 8TB HDD
    -   Seagate BarraCuda 8TB HDD
    -   Seagate ST8000AS0002 Archive v2 8TB HDD
    -   Western Digital WD5000AAKX Caviar 500GB HDD (CCTV)
-   **PCIe Devices**
    -   **PCIe1:** ASUS XG-C100C (Aquantia AQC107) 10Gbe Network Card
-   **USB Devices**
    -   **USB1:** Sandisk Ultra USB 3.0 (Unraid License)
    -   **USB2:** Corsair Link (Corsair RMi 1000w PSU USB)
    -   **USB3:** Ricoh SP211 USB Laser Printer
    -   **USB4:** CyberPower VP1200ELCD UPS
-   **Power Supply:** Corsair RMi 1000w (Overkill, but reused)
-   **Case:** Fractal Design Define R5
-   **Average Power Consumption:** 80W (100W with PSU loss)

It's main purpose is for Media Storage, Backups, CCTV Recording and AI

In addition to the main "NAS" system I also have a 3 node Proxmox cluster, all 3 nodes are identical in specification currently.
## Dell Optiplex 3060 Micro (HV1, HV2 and HV3)
![hv.png](/assets/img/old/hv.png)
-   **CPU:** Intel i5-8500T
-   **RAM:** 40GB 3200mt/s (1x8GB & 1x32GB) DDR4 (will replace the 8GB with another 32GB when needed)
-   **M.2 Devices**
    -   Toshiba XG6 256GB NVMe SSD
    -   A+E 2.5G Ethernet Adapter (Realtek 8125B)
-   **SATA SSDs**
    -   Intel Data Center D3-S4510 480GB SSD (Proxmox Host)
-   **Idle Power Consumption:** TBC

This Proxmox cluster mostly runs a Kubernetes cluster and a few other VM's such as Home Assistant, for more details check out the [Virtual Machines](/virtualmachines) and [Kubernetes](/kubernetes) pages.

As for additional equipment, I don't currently have anything special, just a few basic things which are listed below.

## GL.iNet GL-MT6000 WiFi 6 Router
![router.png](/assets/img/old/router.png)
-   **OS:** OpenWRT (https://github.com/pesa1234/MT6000_cust_build)
-   Many firewall rules and VLANs setup to isolate and segregate the network for security.

## POEPlus S1108GTP-SX-SL
-   **2.5Gbe Ports:** 8x RJ45 with POE
-   **10Gbe Ports:** 1x SFP+ with 10Gbe SFP+ RJ45 module (connected to my NAS)

# Other Equipment
## Cyberpower VP1200ELCD UPS
![ups.png](/assets/img/old/ups.png)
-   **Battery Backup Sockets:** 4x IEC
-   **Surge Only Sockets:** 4x IEC
-   **Power Capacity:** 1200VA / 720W
-   **Estimated Runtime:** 40 Minutes
-   **Average Load:** 25%
-   Configured in Unraid with NUT (Network UPS Tools)

## Bambulab A1 3D Printer
![a1.png](/assets/img/old/a1.png)
-   **AMS Lite**
-   **TP Link Tapo Camera**
-   **ESPHome Smart Plug**

## Ricoh SP211 Mono Laser Printer
![sp211.png](/assets/img/old/sp211.png)
-   **Connectivity:** USB to CUPS Docker Container on NAS