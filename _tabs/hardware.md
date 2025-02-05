---
icon: fas fa-server
order: 1
layout: page
---

Here is a summary of my hardware in use for my homelab.

I currently run a Proxmox cluster of 3 Dell Optiplex 3060's that i purchased for cheap on ebay along with a custom built tower running UnRaid as my NAS

The NAS system is a custom built PC using consumer parts, recently i replaced the aging X299 platform with a AMD AM4 based one.

I saw a good deal on 2 Optiplex 3060 Micro's from ebay which i couldn't ignore so i decided to cluster these 2 systems together using Proxmox. 
![image.png](/assets/img/old/image.png)

After a few months I grabbed another Optiplex 3060 making my cluster Proxmox cluster consist of 3 Dell Optiplex 3060's

# Servers

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

## Dell Optiplex 3060 Micro (HV1)
![hv.png](/assets/img/old/hv.png)
-   **CPU:** Intel i5-8500T
-   **RAM:** 40GB 3200mt/s (1x8GB & 1x32GB) DDR4 (will replace the 8GB with another 32GB when needed)
-   **M.2 Devices**
    -   Toshiba XG6 256GB NVMe SSD
    -   A+E 2.5G Ethernet Adapter (Realtek 8125B)
-   **SATA SSDs**
    -   Intel Data Center D3-S4510 480GB SSD (Proxmox Host)
-   **Idle Power Consumption:** TBC

## Dell Optiplex 3060 Micro (HV2)
![hv.png](/assets/img/old/hv.png)
-   **CPU:** Intel i5-8500T
-   **RAM:** 40GB 3200mt/s (1x8GB & 1x32GB) DDR4 (will replace the 8GB with another 32GB when needed)
-   **M.2 Devices**
    -   Toshiba XG6 256GB NVMe SSD
    -   A+E 2.5G Ethernet Adapter (Realtek 8125B)
-   **SATA SSDs**
    -   Intel Data Center D3-S4510 480GB SSD (Proxmox Host)
-   **Idle Power Consumption:** TBC

## Dell Optiplex 3060 Micro (HV3)
![hv.png](/assets/img/old/hv.png)
-   **CPU:** Intel i5-8500T
-   **RAM:** 40GB 3200mt/s (1x8GB & 1x32GB) DDR4 (will replace the 8GB with another 32GB when needed)
-   **M.2 Devices**
    -   Toshiba XG6 256GB NVMe SSD
    -   A+E 2.5G Ethernet Adapter (Realtek 8125B)
-   **SATA SSDs**
    -   Intel Data Center D3-S4510 480GB SSD (Proxmox Host)
-   **Idle Power Consumption:** TBC

# Network Equipment
## GL.iNet GL-MT6000 WiFi 6 Router
![router.png](/assets/img/old/router.png)
-   **OS:** OpenWRT (https://github.com/pesa1234/MT6000_cust_build)

## POEPlus S1108GTP-SX-SL
-   **2.5Gbe Ports:** 8x RJ45 with POE
-   **10Gbe Ports:** 1x SFP+ with 10Gbe SFP+ RJ45 module

# Other Equipment
## Cyberpower VP1200ELCD UPS
![ups.png](/assets/img/old/ups.png)
-   **Battery Backup Sockets:** 4x IEC
-   **Surge Only Sockets:** 4x IEC
-   **Power Capacity:** 1200VA / 720W
-   **Estimated Runtime:** 40 Minutes
-   **Average Load:** 25%

## Bambulab A1 3D Printer
![a1.png](/assets/img/old/a1.png)
-   **AMS Lite**
-   **TP Link Tapo Camera**
-   **ESPHome Smart Plug**

## Ricoh SP211 Mono Laser Printer
![sp211.png](/assets/img/old/sp211.png)
-   **Connectivity:** USB to CUPS Docker Container on NAS