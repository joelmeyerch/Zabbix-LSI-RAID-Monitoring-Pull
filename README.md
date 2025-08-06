# Zabbix template for Intel/LSI/Symbios RAID Controllers (Pull-Only)

[Topic](https://www.zabbix.com/forum/showthread.php?t=41439) on zabbix forum

**I have converted this Template to pull-only monitoring. The Zabbix server/proxy pulls all data directly from the monitored hosts via Zabbix agent so eliminating the need for a connection from Client to Zabbix server.**

## LSI_RAID_valuemaps
Zabbix value mappings for "Template LSI RAID". Should be imported before template via "Administration" -> "General" -> "Value mapping" -> "Import".

### Available items:
**LSI RAID BBU & LD Status**
- 0 -> Optimal
- 1 -> Failed
 
**LSI RAID PhysDrv Status**
- 0 -> (Online|Hostpare|Unconfigured good)
- 1 -> Failed
- 2 -> Rebuild

## LSI_RAID_template
Zabbix template "Template LSI RAID (pull-only)". Should be imported after Zabbix value mappings via "Configuration" -> "Templates" -> "Import".

### Available items:
**Adapter**
- Adapter model
- Firmware version

**Battery Backup Unit**
- BBU State (+trigger)
- BBU State of charge (+trigger)
- BBU manufacture date (disabled by default)
- BBU design capacity (disabled by default)
- BBU current capacity (disabled by default)

**Physical drives**
- Firmware state (+trigger)
- Predictive errors (+trigger)
- Media errors (+trigger)
- Size
- Model

**Logical volumes**
- Volume state (+trigger)
- Volume size

## Scripts
2 scripts are available for windows (powershell) and unix (perl) servers.

### RAID Discovery script
This script is used for Low Level Discovery of RAID configuration. The script returns JSON data directly when called by Zabbix agent. Discovery runs every hour by default.

### RAID checks script
This script is used by zabbix agent to check all RAID items including critical status information like BBU state, physical drive states, errors, and logical volume states.

## Agent userparameters:

### Windows

    UserParameter=hw.raid.discovery.adapters,C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_discovery.ps1" -DiscoveryType adapters
    UserParameter=hw.raid.discovery.bbu,C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_discovery.ps1" -DiscoveryType bbu
    UserParameter=hw.raid.discovery.pdisks,C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_discovery.ps1" -DiscoveryType pdisks
    UserParameter=hw.raid.discovery.vdisks,C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_discovery.ps1" -DiscoveryType vdisks
    UserParameter=hw.raid.physical_disk[*],C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_check.ps1" -mode pdisk -item $4 -adapter $1 -enc $2 -pdisk $3
    UserParameter=hw.raid.logical_disk[*],C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_check.ps1" -mode vdisk -item $3 -adapter $1 -vdisk $2
    UserParameter=hw.raid.bbu[*],C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_check.ps1" -mode bbu -item $2 -adapter $1
    UserParameter=hw.raid.adapter[*],C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File "C:\Program Files\zabbix_agent\raid_check.ps1" -mode adapter -item $2 -adapter $1

### Unix/Linux

    UserParameter=hw.raid.discovery.adapters,/usr/bin/perl -w /etc/zabbix/scripts/raid_discovery_adapters.pl
    UserParameter=hw.raid.discovery.bbu,/usr/bin/perl -w /etc/zabbix/scripts/raid_discovery_bbu.pl
    UserParameter=hw.raid.discovery.pdisks,/usr/bin/perl -w /etc/zabbix/scripts/raid_discovery_pdisks.pl
    UserParameter=hw.raid.discovery.vdisks,/usr/bin/perl -w /etc/zabbix/scripts/raid_discovery_vdisks.pl
    UserParameter=hw.raid.physical_disk[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode pdisk -item $4 -adapter $1 -enclosure $2 -pdisk $3
    UserParameter=hw.raid.logical_disk[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode vdisk -item $3 -adapter $1 -vdisk $2
    UserParameter=hw.raid.bbu[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode bbu -item $2 -adapter $1
    UserParameter=hw.raid.adapter[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode adapter -item $2 -adapter $1

For agents on unix servers RAID tool should be executed via sudo, add this to sudoers file:

    Defaults:zabbix !requiretty
    # path to your tool can be different
    zabbix  ALL=NOPASSWD:/opt/MegaRAID/CmdTool2/CmdTool2


I'm not a programmer, so code review will be appreciated :)
