---
icon: mdi mdi-arm-flex
order: 30
layout: page
---

# Proxmox
Proxmox Virtual Environment is a hyper-converged infrastructure open-source software. It is a hosted hypervisor that can run operating systems including Linux and Windows on x64 hardware.

I have 3 nodes in my [**Proxmox VE**](https://www.proxmox.com/en/proxmox-ve) Cluster, these are all [Dell Optiplexes](/hardware). This allows for advanced functionality like clustering, replication and utilizing Proxmox Backup Server

## Tips & Tricks
### Shrink Virtual Disk on ZFS Volume
Use Gparted in the VM to resize the partitions (i'd recommend smaller than desired for now, like 90GB for 100GB etc.)

In PVE Console run `zfs set volsize=XXXG rpool/data/vm-XXX-disk-X` replacing X's with the relevant value then run `qm rescan`

In Gparted ignore the partition table warning and open terminal
`sudo gdisk /dev/sdX`
`x` `e` `p` `w` 

Rescan in Gparted and expand previously shrunk partition to use the remaining disk.

https://t.du9l.com/2023/12/shrinking-the-root-disk-of-a-proxmox-ve-virtual-machine/

### Revert clustered node to solo host

```bash
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
rm /etc/pve/corosync.conf
rm /etc/corosync/*
killall pmxcfs
systemctl start pve-cluster
```
https://www.reddit.com/r/Proxmox/comments/avk2gx/help_cluster_not_ready_no_quorum_500/

### Converting Proxmox Legacy ZFS install to UEFI ZFS
After trial and error and mixing and matching many guides together i struggled but managed to successfully convert my legacy ZFS install using GRUB to a UEFI ZFS install using systemd-boot, after much googling i found that proxmox-boot-tool does not install EFI files unless you are booted in EFI, this means we have to boot to the Proxmox ISO in UEFI mode to get the UEFI files installed. Here are the steps i did.

1. Ensure your Proxmox is relatively up to date before proceeding
2. Ensure systemd-boot is installed, trust me this is a pain in the ass if you forget this `apt-get install systemd-boot`
3. Boot using a Proxmox VE version 6.4 or newer ISO in **UEFI MODE** (CSM Disabled)
4. Select Install Proxmox VE (Console Debug)
5. Exit the first debug shell by typing `Ctrl + D` or exit. The second debug shell contains all the necessary binaries for the following steps
7. Import the root pool (usually named rpool) with an alternative mountpoint of /mnt:
```bash
zpool import -f -R /mnt rpool
```
9. Bind-mount all virtual filesystems needed for running proxmox-boot-tool:
```bash
mount -o rbind /proc /mnt/proc
mount -o rbind /sys /mnt/sys
mount -o rbind /dev /mnt/dev
mount -o rbind /run /mnt/run
```
10. change root into /mnt
```bash
chroot /mnt /bin/bash
```

11. Find the spare "Fat32" partitions you can use, in this example they are /dev/sda2, /dev/sdb2 and /dev/sdc2
[Finding_potential_ESPs](https://pve.proxmox.com/wiki/ZFS:_Switch_Legacy-Boot_to_Proxmox_Boot_Tool#3._Finding_potential_ESPs)

12. Replace "ID" with your partition identifier (e.g sda2, sdb2, sdc3) and run the following command, if there is already an existing partition you can append `--force` be careful<br />repeat this step for each ESP partition you will create (in this example it is 3 as per the picture)
```bash
proxmox-boot-tool format /dev/ID
```

13. Run the following command again replacing ID with your partition identifier(s)<br />repeat this step for each ESP partition you will create (in this example it is 3 as per the picture)
```bash
proxmox-boot-tool init /dev/ID
```

14. you can verify the EFI boot entries with `efibootmgr -v`.

15. Exit the chroot-shell (`Ctrl + D` or exit) and reset the system (for example by pressing `CTRL + ALT + DEL`)

16. You should now be able to boot to your Proxmox installation with CSM disabled with the `Linux Boot Manager` option in your boot list

#### References:
[Repairing_a_System_Stuck_in_the_GRUB_Rescue_Shell](https://pve.proxmox.com/wiki/ZFS:_Switch_Legacy-Boot_to_Proxmox_Boot_Tool#Repairing_a_System_Stuck_in_the_GRUB_Rescue_Shell)
[Finding_potential_ESPs](https://pve.proxmox.com/wiki/ZFS:_Switch_Legacy-Boot_to_Proxmox_Boot_Tool#3._Finding_potential_ESPs)