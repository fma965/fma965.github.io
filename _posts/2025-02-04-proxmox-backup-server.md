---
layout: post
title: ""
date: 2025-02-05 10:00:00 0000
categories: homelab services backup
tags: homelab backup proxmox pbs

---

## [Proxmox Backup Server](https://www.proxmox.com/en/proxmox-backup-server/overview)
### Tabs {.tabset}
#### Info
Proxmox Backup Server is an enterprise backup solution, for backing up and restoring VMs, containers, and physical hosts. By supporting incremental, fully deduplicated backups, Proxmox Backup Server significantly reduces network load and saves valuable storage space. With strong encryption and methods of ensuring data integrity, you can feel safe when backing up data, even to targets which are not fully trusted.

Proxmox Backup Server stores backups for my online services (not mentioned here) aswell as backups of all my important local homelab VM's but not only that but it also stores backups of the /etc and /root folders of all my Proxmox hosts (script below)

#### PVE Host (Linux) Backup Script
cronjob - runs at midnight each day (hour incremented for each node)
```bash
0 0 * * * /bin/bash /root/pbs-backup.sh > /dev/null 2>&1
```
/root/pbs-backup.sh
```bash
#!/bin/bash
export PBS_PASSWORD="[REDACTED]"
#export PBS_LOG=warn
proxmox-backup-client backup \
        pve-etc.pxar:/etc \
        pve-root.pxar:/root \
        --exclude *.vscode-server \
        --repository f9@pbs@[REDACTED]:unraid-pbs \
        --ns f9 \
        --include-dev /etc/pve \
        --skip-lost-and-found
```

Proxmox Backup Server currently runs as a Docker container on my UnRAID machine.