# SR linux upgrade/downgrade procedure  20.6.6 <-> 24.10.1

In this guide, we'll demonstrate how to upgrade from version 20.6.6 to 24.10.1. We'll cover two methods: a manual upgrade and an automated process using ZTP (Zero Touch Provisioning).

## Manual upgrade (20.6.6 -> 24.10.1)
1. Download the image to the /tmp directory (for example, using HTTP):

```
A:srlinux# bash
bash-4.2$ ip netns exec srbase-mgmt bash
bash-4.2$ cd /tmp/
bash-4.2$ curl -O http://135.227.248.166/load/24.10.1-195/srlinux-24.10.1-195.bin
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1433M  100 1433M    0     0  44.2M      0  0:00:32  0:00:32 --:--:-- 36.1M
```
2. Extract the SR Linux image:

```
bash-4.2$ sudo mkdir -p /mnt/nokiaos/24.10.1-195
bash-4.2$ sudo chmod +x /tmp/srlinux-24.10.1-195.bin
bash-4.2$  sudo /tmp/srlinux-24.10.1-195.bin --target /mnt/nokiaos/24.10.1-195 --noexec
Creating directory /mnt/nokiaos/24.10.1-195
Verifying archive integrity...  100%   MD5 checksums are OK. All good.
Uncompressing srlinux-24.10.1-195:release  100%
```
3. Initiate an SR Linux CLI session and check the current software version:


```
bash-4.2$ sr_cli
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ running }--[  ]--
A:srlinux# info from state system boot image
    system {
        boot {
            image [
                20.6.6-19
            ]
        }
    }
```
4. Update the boot image list by placing the current version after the new version:

```
A:srlinux# enter candidate
--{ candidate shared default }--[  ]--
A:srlinux# system boot image [ 24.10.1-195 20.6.6-19 ]
--{ * candidate shared default }--[  ]--
A:srlinux# commit stay save
All changes have been committed. Starting new transaction.
/system:
    Saved current running configuration as initial (startup) configuration '/etc/opt/srlinux/config.json'

--{ candidate shared default }--[  ]--
```
5. Reboot the chassis:

```
A:srlinux# tools platform chassis reboot
```
6. Check the new version is applied:

```
A:usgry1-123-02-13-shr1-gw2# show version
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Hostname             : usgry1-123-02-13-shr1-gw2
Chassis Type         : 7250 IXR-10
Part Number          : 3HE11770AARA01
Serial Number        : NS1946F0070
System HW MAC Address: 40:55:82:B0:89:D9
OS                   : SR Linux
Software Version     : v24.10.1
Build Number         : 195-gd59dc3427c
Architecture         : x86_64
Last Booted          : 2024-10-28T13:27:16.436Z
Total Memory         : 15898726 kB
Free Memory          : 12070968 kB
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
--{ [OLD RESCUE] candidate shared default }--[  ]--
A:usgry1-123-02-13-shr1-gw2#
```
## Manual downgrade (24.10.1 -> 20.6.6)
*Note*: The current running configuration is version 24.10.1, while the startup configuration remains at 20.6.6. If you save the running configuration as the startup configuration, downgrading the image will not be supported. Please create a backup of your configuration before saving the running configuration to startup.

1. Upgrade using a HTTP/HTTPS link (In the following example, the reboot option is not used. After the image is deployed, the system must be rebooted separately.):

```
A:usgry1-123-02-13-shr1-gw2# tools system deploy-image http://135.227.248.166/load/20.6.6-19/srlinux-20.6.6-19.bin
Downloading with the srbase-mgmt namespace. Connection timeout: 5 seconds
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  780M  100  780M    0     0  33.1M      0  0:00:23  0:00:23 --:--:-- 29.5M
Extracting the version of the image file
Deploying SRL image version 20.6.6-19
2024:10:28 13:51:00:58 | EVENT | Installing image. Url: file:///tmp/srlinux-20.6.6-19.bin
2024:10:28 13:51:03:47 | EVENT | Version of new image 20.6.6-19
2024:10:28 13:51:03:47 | EVENT | Current version: 24.10.1-195, New version: 20.6.6-19
2024:10:28 13:51:57:80 | EVENT | Syncing image with standby
Successfully deployed SRL image version 20.6.6-19
--{ [OLD RESCUE] candidate shared default }--[  ]--
A:usgry1-123-02-13-shr1-gw2#
```
2. Reboot the node:
```
A:usgry1-123-02-13-shr1-gw2# tools platform chassis reboot
```

## Automatic upgrade via ZTP (20.6.6 -> 24.10.1)
In this procedure, we will enable ZTP functionality and perform a node upgrade using ZTP.

1. Turn on ZTP

```
A:srlinux# enter candidate
--{ [FACTORY] candidate shared default }--[  ]--
A:srlinux# system boot autoboot admin-state enable
--{ [FACTORY] * candidate shared default }--[  ]--
A:srlinux# commit stay save
All changes have been committed. Starting new transaction.
/system:
    Saved current running configuration as initial (startup) configuration '/etc/opt/srlinux/config.json'

--{ candidate shared default }--[  ]--
A:srlinux#
```
2. Reboot the node:

```
A:srlinux# tools platform chassis reboot
```
3. Monitor the ZTP logs during uptime (please note that the configuration is in the 20.6.6 format):

```
SRLINUX 20.6.6-19
Kernel 4.19.39-2.x86_64 on an x86_64

srlinux login: 2024:10:28 14:48:02:02 | EVENT | Starting ZTP process
2024:10:28 14:48:02:05 | EVENT | Current version 20.6.6-19
2024:10:28 14:48:08:88 | EVENT | Set link mgmt0 up
2024:10:28 14:48:13:95 | EVENT | ZTP runtime remaining 3588.04 seconds
2024:10:28 14:48:13:96 | EVENT | ZTP Perform DHCP_V4. attempt[0]
2024:10:28 14:48:14:32 | EVENT | Received dhcp lease on mgmt0 for 135.227.254.27/21
2024:10:28 14:48:14:36 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:48:22:49 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:48:22:50 | EVENT | Url to fetch provisioning script: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:48:22:50 | EVENT | Executing provisioning script
2024:10:28 14:48:22:51 | EVENT | Fetching http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:48:22:58 | EVENT | Copying ['http://135.227.248.166/ztp/ztp-ixr10-1.py'] to /tmp/ztp/tmppx18v14bztp-ixr10-1.py
2024:10:28 14:48:22:58 | EVENT | Downloaded provisioning script to /etc/opt/srlinux/ztp/script/provision.py
2024:10:28 14:48:23:34 | EVENT | Fetching http://135.227.248.166/load/24.10.1-195/srlinux-24.10.1-195.bin.md5
2024:10:28 14:48:23:38 | EVENT | Copying ['http://135.227.248.166/load/24.10.1-195/srlinux-24.10.1-195.bin.md5'] to /tmp/ztp/srlinux-upgrade-dir/srlinux-24.10.1-195.bin.md5
2024:10:28 14:48:23:38 | EVENT | Fetching http://135.227.248.166/load/24.10.1-195/srlinux-24.10.1-195.bin
2024:10:28 14:48:55:33 | EVENT | Copying ['http://135.227.248.166/load/24.10.1-195/srlinux-24.10.1-195.bin'] to /tmp/ztp/srlinux-upgrade-dir/srlinux-24.10.1-195.bin
2024:10:28 14:49:19:85 | EVENT | Version of new image 24.10.1-195
2024:10:28 14:49:19:85 | EVENT | Current version: 20.6.6-19, New version: 24.10.1-195
2024:10:28 14:49:24:25 | EVENT | Invoked sync call. It may take few seconds to complete.
2024:10:28 14:50:36:65 | EVENT | Syncing image with standby
2024:10:28 14:50:45:70 | EVENT | Copied image to standby
2024:10:28 14:50:59:69 | EVENT | Reboot will be triggered
```
4.Verify that the configuration has been applied and that the version is 24.10.1:


```
SRLINUX 24.10.1-195
Kernel 6.1.25-28-amd64 on an x86_64

usgry1-123-02-13-shr1-gw2 login: 2024:10:28 14:54:22:21 | EVENT | Starting ZTP process
2024:10:28 14:54:22:23 | EVENT | Current version 24.10.1-195
2024:10:28 14:54:41:80 | EVENT | ZTP runtime remaining 3580.38 seconds
2024:10:28 14:54:41:81 | EVENT | ZTP Perform DHCP_V4
2024:10:28 14:54:42:02 | EVENT | Received dhcp lease on mgmt0 for 135.227.254.27/21, from server 135.227.248.166
2024:10:28 14:54:42:08 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:54:42:09 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:54:42:10 | EVENT | Url to fetch provisioning script: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 14:54:42:10 | EVENT | Executing provisioning script
2024:10:28 14:54:42:14 | EVENT | Downloaded provisioning script to /etc/opt/srlinux/ztp/script/provision.py
2024:10:28 14:54:44:23 | EVENT | Starting srlinux
2024:10:28 14:54:48:73 | EVENT | Srlinux process state: not-running
2024:10:28 14:55:06:06 | EVENT | Srlinux is running
2024:10:28 14:55:15:51 | EVENT | Execution of /etc/opt/srlinux/ztp/script/provision.py completed with exit code 0
2024:10:28 14:55:15:52 | EVENT | Provisioning script execution successful
2024:10:28 14:55:24:37 | EVENT | Completed ZTP process

usgry1-123-02-13-shr1-gw2 login: linuxadmin
Password:

[linuxadmin@usgry1-123-02-13-shr1-gw2 ~]$ sr_cli
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ [OLD RESCUE] running }--[  ]--
A:usgry1-123-02-13-shr1-gw2# show version
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Hostname             : usgry1-123-02-13-shr1-gw2
Chassis Type         : 7250 IXR-10
Part Number          : 3HE11770AARA01
Serial Number        : NS1946F0070
System HW MAC Address: 40:55:82:B0:89:D9
OS                   : SR Linux
Software Version     : v24.10.1
Build Number         : 195-gd59dc3427c
Architecture         : x86_64
Last Booted          : 2024-10-28T14:53:32.335Z
Total Memory         : 15898726 kB
Free Memory          : 11403011 kB
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
--{ [OLD RESCUE] running }--[  ]--
A:usgry1-123-02-13-shr1-gw2# 24-10-28 14:57:04.921 sr_device_mgr: Standby is now synchronized
```
## Automatic downgrade via ZTP (24.10.1 ->  20.6.6 )
This procedure will closely resemble the upgrade process; however, at this stage, we will reference version 20.6.6 in the ZTP scripts.
1. Enable autoboot:

```
A:usgry1-123-02-13-shr1-gw2# enter candidate
--{ [OLD RESCUE] candidate shared default }--[  ]--
A:usgry1-123-02-13-shr1-gw2# system boot autoboot admin-state enable
--{ [OLD RESCUE] * candidate shared default }--[  ]--
A:usgry1-123-02-13-shr1-gw2# commit stay save
/system:
    Saved current running configuration as initial (startup) configuration '/etc/opt/srlinux/config.json'

All changes have been committed. Starting new transaction.
--{ [OLD RESCUE] candidate shared default }--[  ]--
A:usgry1-123-02-13-shr1-gw2#
```
2. Reboot the node:

```
A:usgry1-123-02-13-shr1-gw2# tools platform chassis reboot
24-10-28 14:59:51.210 sr_device_mgr: Chassis reboot requested - rebooting
```
3. Review the ZTP logs during uptime (please note that the configuration is in the 20.6.6 format):

```
SRLINUX 24.10.1-195
Kernel 6.1.25-28-amd64 on an x86_64

usgry1-123-02-13-shr1-gw2 login: 2024:10:28 15:01:42:48 | EVENT | Starting ZTP process
2024:10:28 15:01:42:49 | EVENT | Current version 24.10.1-195
2024:10:28 15:02:01:38 | EVENT | ZTP runtime remaining 3581.06 seconds
2024:10:28 15:02:01:39 | EVENT | ZTP Perform DHCP_V4
2024:10:28 15:02:01:60 | EVENT | Received dhcp lease on mgmt0 for 135.227.254.27/21, from server 135.227.248.166
2024:10:28 15:02:01:65 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:02:01:66 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:02:01:67 | EVENT | Url to fetch provisioning script: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:02:01:67 | EVENT | Executing provisioning script
2024:10:28 15:02:01:69 | EVENT | Downloaded provisioning script to /etc/opt/srlinux/ztp/script/provision.py
2024:10:28 15:02:26:12 | EVENT | Installing image. Url: http://135.227.248.166/load/20.6.6-19/srlinux-20.6.6-19.bin
2024:10:28 15:02:31:80 | EVENT | Version of new image 20.6.6-19
2024:10:28 15:02:31:80 | EVENT | Current version: 24.10.1-195, New version: 20.6.6-19
2024:10:28 15:03:27:79 | EVENT | Syncing image with standby
2024:10:28 15:04:47:63 | EVENT | Reboot will be triggered
2024:10:28 15:04:47:79 | EVENT | CPM card reboot requested.
2024:10:28 15:04:47:83 | EVENT | reboot is called. Output  :
```
4. Ensure that the configuration has been applied and that the version is 20.6.6:


```
SRLINUX 20.6.6-19
Kernel 4.19.39-2.x86_64 on an x86_64

localhost login: 2024:10:28 15:06:23:00 | EVENT | Starting ZTP process
2024:10:28 15:06:23:03 | EVENT | Current version 20.6.6-19
2024:10:28 15:06:30:06 | EVENT | Set link mgmt0 up
2024:10:28 15:06:35:10 | EVENT | ZTP runtime remaining 3587.87 seconds
2024:10:28 15:06:35:13 | EVENT | ZTP Perform DHCP_V4. attempt[0]
2024:10:28 15:06:35:43 | EVENT | Received dhcp lease on mgmt0 for 135.227.254.27/21
2024:10:28 15:06:35:48 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:06:43:61 | EVENT | option 67 provided by dhcp: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:06:43:61 | EVENT | Url to fetch provisioning script: http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:06:43:61 | EVENT | Executing provisioning script
2024:10:28 15:06:43:62 | EVENT | Fetching http://135.227.248.166/ztp/ztp-ixr10-1.py
2024:10:28 15:06:43:67 | EVENT | Copying ['http://135.227.248.166/ztp/ztp-ixr10-1.py'] to /tmp/ztp/tmpi_1jkcf4ztp-ixr10-1.py
2024:10:28 15:06:43:68 | EVENT | Downloaded provisioning script to /etc/opt/srlinux/ztp/script/provision.py
2024:10:28 15:06:45:24 | EVENT | Fetching http://135.227.248.166/configs/ztp/ixr10-1-20_6_6.json
2024:10:28 15:06:45:25 | EVENT | Copying ['http://135.227.248.166/configs/ztp/ixr10-1-20_6_6.json'] to /tmp/ztp/tmp5que6jrhixr10-1-20_6_6.json
2024:10:28 15:06:45:26 | EVENT | Starting srlinux
2024:10:28 15:07:00:40 | EVENT | Srlinux process state: not-running
2024:10:28 15:07:06:50 | EVENT | Srlinux is running

SRLINUX 20.6.6-19
Kernel 4.19.39-2.x86_64 on an x86_64

A:usgry1-123-02-13-shr1-gw2# show version
--------------------------------------------------------------------------------
Hostname          : usgry1-123-02-13-shr1-gw2
Chassis Type      : 7250 IXR-10
Part Number       : 3HE11770AARA01
Serial Number     : NS1946F0070
System MAC Address: 40:55:82:B0:89:D9
Software Version  : v20.6.6
Build Number      : 19-g26361862fa
Architecture      : x86_64
Last Booted       : 2024-10-28T15:06:59.800Z
Total Memory      : 15914664 kB
Free Memory       : 11594359 kB
--------------------------------------------------------------------------------
--{ + running }--[  ]--
A:usgry1-123-02-13-shr1-gw2#
```
