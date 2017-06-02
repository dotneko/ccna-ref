### CDP Configuration and Verification

- CDP is enabled by default on Cisco devices.
- Desirable to disable CDP on a network device globally or per interface.

```
show cdp                # Displays CDP status
show cdp neighbors      # Shows information about CDP neighbors
show cdp neighbors detail
show cdp interface

cdp run                 # Enable CDP globally
no cdp run              # Disable CDP globally

interface <interface-id>
  no cdp enable   # Disable CDP on specific interface
```

### LLDP Configuration and Verification

- LLDP must be configured separately to transmit and receive LLDP packets.
```
lldp run
interface <interface-id>
  lldp transmit
  lldp receive
  exit
  
show lldp
show lldp neighbors
show lldp neighbors detail
```

### `clock` Command

```
clock set hh:mm:ss mmm dd yyyy

clock set 20:36:00 dec 12 2015
clock timezone <timezone>
clock summer-time PDT recurring

```

### NTP Configuration and Verification

```
show clock              # Displays current time on software clock
show clock detail       # Displays additionally the time source
```

Configure NTP Server

```
ntp server <ntp-server-ip>
```

Verify configuration

```
show clock detail
show ntp associations
show ip ntp associations
show ntp status
```

### Default Logging

Enable logging

```
logging console			# Enables logging to console
logging buffered		# Enables logging to buffer

show logging			# Shows default logging service and messages in buffer
```

- In most Cisco IOS routers, default severity level is 7, debugging.

### Syslog Configuration

Three steps to configure router to send system messages to syslog server:

1. `logging <ip-address>` to configure desination of syslog server.
2. `logging trap <level>` to control messages that will be sent.
3. `logging source-interface <interface-id>` specifies syslog packets that contain the IPv4/IPv6 address of the interface will be logged.

Example (Tftpd32 syslog server setup on 192.168.1.3):

```
logging 192.168.1.3
logging trap 4
logging source-interface g0/0

interface loopback 0            # Force messages via loopback
shutdown
no shutdown
```

### Verify Syslog

Examples:

```
show logging | include changed state to up
show logging | begin June 12 22:35
```

### Enable Logging Timestamps

```
service timestamps log datetime msec
```

### Router File Systems

Cisco IOS File System (IFS)

```
show file systems
```

- Current file system is indicated by preceding asterisk.
- Pound symbol (#) is appended to the bootable disk.

Flash File System

```
dir
```

NVRAM File System

```
cd nvram:
pwd
dir
```

### Backing up and Restoring TFTP

Backup:

```
copy running-config tftp
copy startup-config tftp
```

Restore:

```
copy tftp running-config
copy tftp startup-config
```

### Using USB Ports on a Cisco Router

```
dir usbflash0:
copy running-config usbflash0:/
copy usbflash:0/<filename> running-config
more usbflash0:/<filename>
```

### Password Recovery

1. Enter ROMMON mode
2. `conf-reg 0x2142` to ignore startup-config
3. Make necessary changes to original startup-config
4. Save the new configuration
5. `config-register 0x2102` to re-enable startup-config
6. `reset`

# License Verification and Management

### License Installation and Backup

Determine the UDI on router:

```
show license udi
```

Installing a license:

```
license install <stored-location-url>
reload
```

License verification:

```
show version
show license
```

Configure a one-time acceptance of the EULA:

```
license accept end user agreement
```

Activate an Evaluation RTU license:

```
license boot module <module-name> technology-package <package-name>
reload              # Required to activate software package
```

Technology package names for Cisco ISR G2:

- ipbasek9
- securityk9
- datak9
- uck9

#### Backup the license

```
license save <location:filename.lic>
```

### Removing a license
To clear an active permanent license from the Cisco 1900 series, 2900 series, and 3900 series routers, perform the following steps:

Step 1. Disable the technology package active license with the command:

```
license boot module module-name technology-package package-name disable
```

Reload the router using the `reload` command. A reload is required to make the software package inactive.

Step 2. Clear the license from license storage:

```
license clear feature-name
```

Clear the license boot module command used for disabling the active license:
```
no license boot module <module-name> technology-package <package-name> disable
```
Note: Some licenses, such as built-in licenses, cannot be cleared. Only licenses that have been added by using the license install command are removed. Evaluation licenses are not removed.
