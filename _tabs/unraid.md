---
icon: mdi mdi-server
order: 40
layout: page
---

Below you can find details of everything that is currently running on my Unraid Server. 

For docker containers refer to the [Service Categories](/categories)

For details on the hardware refer to [Hardware](/hardware)

# F9-NAS
- **Unraid Version:** 7.0.0 

## Plugins
- [Appdata Backup *A backup plugin for your docker container templates, its appdata and some more.*](https://forums.unraid.net/topic/137710-plugin-appdatabackup/)
- [CA Auto Update Applications *Part of the CA family, Auto Update Applications will keep your selected plugins and docker applications up to date*](https://forums.unraid.net/topic/51959-plugin-ca-application-auto-update/)
- [CA Cleanup Appdata *Scan your appdata share to determine which folders are no longer in use and prompt you to remove them.*](https://forums.unraid.net/topic/51961-plugin-ca-cleanup-appdata/)
- [Community Applications *The One Stop Shop for all applications for Unraid*](https://forums.unraid.net/topic/38582-plug-in-community-applications)
- [Compose Manager *This plugin installs the docker compose package on you unRAID server and adds a simple manager page to the web ui.*](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager/)
- [CoreFreq *A CPU low level monitoring software designed for x86_64-Processors*](https://forums.unraid.net/topic/109314-plugin-corefreq/)
- [Corsair PSU Statistics *Reads the statistics from Corsair PSU's with a USB port and displays them in the dashboard.*](https://forums.unraid.net/topic/86715-corsair-rmi-hxi-axi-psu-statistics-cyanlabss-fork/)
- [Dynamix Active Streams *Shows in real-time any open SMB, AFP and Plex network streams.*](https://forums.unraid.net/index.php?topic=36543.0)
- [Dynamix System Buttons *Ddds an one-click button to the header which allows for instant sleep, reboot or shutdown of the system.*](https://forums.unraid.net/index.php?topic=36543.0)
- [Dynamix System Temperature *Shows in real-time the temperature of the system CPU and motherboard.*](https://forums.unraid.net/index.php?topic=36543.0)
- [Enhanced Log Viewer *View the system log with highlighted lines. event highlighting and your own colors for each event.*](https://forums.unraid.net/topic/43115-enhanced-log-view-with-lines-highlighted-in-color/)
- [Fix Commong Problems *Diagnose and suggest fixes for common problems, configuration mistakes, etc.*](https://forums.unraid.net/topic/47266-plugin-ca-fix-common-problems/)
- [FolderView *Create folders for grouping Dockers and VMs together to help with organization.*](https://forums.unraid.net/topic/142782-plugin-folderview/)
- [GPU Statistics *Display GPU statistics on the dashboard*](https://forums.unraid.net/topic/89453-plugin-gpu-statistics/)
- [Live Memory Tester for UNRAID *Test your RAM without needing to reboot.*](https://forums.unraid.net/topic/168125-plugin-live-memory-tester-for-unraid/)
- [Network UPS Tools (NUT) for UNRAID *Management of your UPS either standalone or as a master/server service*](https://forums.unraid.net/topic/60217-plugin-nut-v2-network-ups-tools/)
- [NVidia Driver *Install the Nvidia drivers to utilize your Nvidia graphics card in your Docker container(s).*](https://forums.unraid.net/topic/98978-plugin-nvidia-driver/)
- [Open Files *Shows any open files on the array that might prevent a clean shutdown.*](https://forums.unraid.net/topic/41196-open-files-plugin-can-help-with-troubleshooting-why-server-wont-shut-down/)
- [Radeon Top *Adds the tool 'radeontop' to your unRAID server and also enables your AMD GPU*](https://forums.unraid.net/topic/92865-support-ich777-nvidiadvb-kernel-helperbuilder-docker/)
- [Tips and Tweaks *Change certain Linux settings that may improve performance of your Unraid server*](https://forums.unraid.net/topic/47527-tips-and-tweaks-plugin-to-possibly-improve-performance-of-unraid-and-vms/?tab=comments#comment-468361)
- [Unassigned Devices *Automount and share disks that are not part of your Unraid array.*](https://forums.unraid.net/topic/92462-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/)
- [Unassigned Devices Plus *Unassigned Devices support for HFS+, exFAT, and apfs disk formats, and enabling destructive mode.*](https://forums.unraid.net/topic/92462-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/)
- [Unraid Patch *Keeps your server current with the latest patches*](https://forums.unraid.net/topic/185560-unraid-patch-plugin/)
- [User Scripts *A simple front end for any user scripts to allow you to run them without entering the command line*](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/)


## Auth Bypass for Corsair Stats and Dynamix Temperature pages
Add the following to the bottom of your "go" file (/boot/config/go)
```bash
# Add Un-Authenticated access to Unraid 6.10-RC1+ for SystemTemp.php and Status.php (Corsair Plugin)
while [ ! -f /var/run/nginx.pid ]
do
  sleep 2 # or less like 0.2
done

echo -e "# Fma965 Un-Authenticated Access\nlocation ~ /plugins\/corsairpsu\/status.php {\nallow all;\ninclude fastcgi_params;\n}\n\nlocation ~ /plugins\/dynamix.system.temp\/include\/SystemTemp.php {\nallow all;\ninclude fastcgi_params;\n}\n# End Fma965 Un-Authenticated Access\n\n$(cat /etc/nginx/conf.d/locations.conf)" > /etc/nginx/conf.d/locations.conf;
nginx -s reload
# End Nginx Basic Auth Patch
```

http://UNRAID_SERVER/plugins/corsairpsu/status.php
http://UNRAID_SERVER/plugins/dynamix.system.temp/include/SystemTemp.php


## Disable AMD-PState-EPP (revert to passive)
Add ` amd_pstate=passive` to the end of your `append initrd=/bzroot` in your /boot/syslinux/syslinux.cfg 

Example
```
default menu.c32
menu title Lime Technology, Inc.
prompt 0
timeout 50
label Unraid OS
  menu default
  kernel /bzimage
  append initrd=/bzroot amd_pstate=passive
label Unraid OS GUI Mode
  kernel /bzimage
  append initrd=/bzroot,/bzroot-gui
label Unraid OS Safe Mode (no plugins, no GUI)
  kernel /bzimage
  append initrd=/bzroot unraidsafemode 
label Unraid OS GUI Safe Mode (no plugins)
  kernel /bzimage
  append initrd=/bzroot,/bzroot-gui unraidsafemode
label Memtest86+
  kernel /memtest
```

## MinIO Issues
MinIO will not work with Unraid's SHFS (Fuse File System), the only solution at this time is to mount a specific disk rather than a user share.

Example instead of `/mnt/user/S3/` we can use a specific disk like `/mnt/disk2/S3/`

I believe ZFS arrays will also work but I have not tested this yet.


### Fix Temperature Sensors Conflict with Corsair and Dynamix
In the User Script plugin add a new script that runs "At Startup of Array" with the following code
```bash
#!/bin/bash

FILE="/etc/sensors.d/sensors.conf"

if [ -f "$FILE" ]; then
    sed -i '/chip "corsairpsu-hid[^"]*"/,+1d' "$FILE"
    echo "Entries have been removed."
else
    echo "$FILE doesn't exist."
fi
```


## Various Settings
### Disk 
- Tunable (md_write_method): reconstruct write

### Docker
- Docker storage driver: native
- Type: Directory
- Preserve user defined networks: Yes
- Host access to custom networks: No

### NUT
- Time on Battery before Shutdown (Minutes): 5
- ups.conf
  ```ini
  override.ups.delay.shutdown = 150
  override.ups.delay.start = 20
  override.ups.beeper.status = "disabled"
  ```

### SMB
- Enable SMB Multi Channel: Yes
- Enable NetBIOS: No

### Tips And Tricks
- Disable NIC Flow Control? Yes
- Disable NIC Offload? Yes
- Normal CPU Scaling Governor: On Demand
- Power Saving CPU Scaling Governor: Power Save
- Power Saving Schedule: 00:00 -> 07:0