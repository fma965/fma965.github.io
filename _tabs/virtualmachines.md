---
icon: mdi mdi-cube-outline
order: 6
layout: page
---

Here is a list of all the Virtual Machines i currently run across my [Proxmox](https://www.proxmox.com/en/proxmox-ve) Cluster consisting of 3 nodes.
## F9-HV1 (Node 1)

### K3S-Master-01

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 10GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Master Node 

### K3S-Worker-01

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 16GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 40GB
-   **Network Device:** vmbr0 (virtio)
-   **PCIe Device:** AMD Onboard GPU

**Purpose**

-   K3S Worker Node 

### K3S-Longhorn-01

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 100GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Longhorn Storage Node 

### F9-GS-01 (LXC)

**Hardware / Resources**

-   **Type:** Container
-   **Memory:** 8GB Fixed
-   **CPUs:** 4 Cores
-   **Hard Disk:** 20GB (Will be increased as needed)
-   **Network Device:** vmbr0

**Purpose**

-   Pterodactyl Game Servers - https://pterodactyl.io/

## F9-HV2 (Node 2)

### K3S-Master-02

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 10GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Master Node 

### K3S-Worker-02

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 16GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 30GB
-   **Network Device:** vmbr0 (virtio)
-   **PCIe Device:** AMD Onboard GPU

**Purpose**

-   K3S Worker Node 

### K3S-Longhorn-02

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 100GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Longhorn Storage Node 

### F9-GS-02 (LXC)

**Hardware / Resources**

-   **Type:** Container
-   **Memory:** 16GB Fixed
-   **CPUs:** 4 Cores
-   **Hard Disk:** 20GB (Will be increased as needed)
-   **Network Device:** vmbr0

**Purpose**

-   Pterodactyl Game Servers - https://pterodactyl.io/

## F9-HV3 (Node 3)

### K3S-Master-03

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 10GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Master Node 

### K3S-Worker-03

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 16GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 30GB
-   **Network Device:** vmbr0 (virtio)
-   **PCIe Device:** AMD Onboard GPU

**Purpose**

-   K3S Worker Node 

### K3S-Longhorn-03

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 4GB Fixed
-   **CPUs:** 4 Cores
-   **BIOS:** SeaBIOS
-   **Machine Type:** i440fx
-   **Hard Disk:** 100GB
-   **Network Device:** vmbr0 (virtio)

**Purpose**

-   K3S Longhorn Storage Node 

### F9-GS-03 (LXC)

**Hardware / Resources**

-   **Type:** Container
-   **Memory:** 8GB Fixed
-   **CPUs:** 4 Cores
-   **Hard Disk:** 20GB (Will be increased as needed)
-   **Network Device:** vmbr0

**Purpose**

-   Pterodactyl Game Servers - https://pterodactyl.io/

### Home Assistant (HassOS)

**Hardware / Resources**

-   **Type:** VM
-   **Memory:** 2 / 3 GB
-   **CPUs:** 2 Cores
-   **BIOS:** OVMF (UEFI)
-   **Machine Type:** i440fx
-   **Hard Disk:** 32GB (raw)
-   **Network Device:** vmbr0 (virtio)
-   **USB Device:** Texas Instruments CC2531 (Zigbee)

**Purpose**

-   Home Automation
    -   ESPHome
    -   Zigbee2MQTT
    -   MQTT
