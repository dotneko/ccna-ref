### Cisco IOS Resilient Features

```
secure boot-image
no secure boot-image

secure boot-config          ! Can be used repeatedly to update config files
no secure boot-config

show secure bootset
```

### Restore a Primary Bootset Image

1. `reload` (If necessary use break sequence to enter ROMmon)
2. `dir` to list contents of device that contains secure bootset.
3. `boot <location:bootsetimage.bin>`
4. `secure boot-config restore <location:filename, e.g. flash0:rescue-cfg>` (GCM#)
5. `copy <location:rescue-cfg> running-config`

### Secure Copy Configuration

```
ip domain-name <domain_name.com>
crypto key generate rsa general-keys modulus 2048
username <name> privilege 15 algorithm-type scrypt secret <password>
aaa new-model
aaa authentication login default local
aaa authorization exec default local
ip scp server enable
```

Using SCP

```
copy flash0:backup-file.cfg scp:
 ! ip-address
 ! username
 ! filename
 ! password
```

Debugging SCP

```
debug ip scp
```

Note: *authentication failure occurs if the username/password combination was not configured with the privilege 15 keyword on the SCP server*.

### Recover Router Passwords

- Connect via console port
- Record the configuration register setting (usually `0x2102`)
- Power cycle the router
- Issue the break sequence
- RomMON: Change default configuration register with `confreg 0x2142` command (bypasses NVRAM)
- Reboot the router
- Press `Ctrl-C` to skip initial setup procedure
- Put router into privileged EXEC mode
- Copy startup configuration to running configuration
- Verify the configuration
- Change the enable secret password
- Enable all interfaces
- Return configuration-register to original setting using `config-register` (Cisco IOS equiv of `confreg`): e.g. `config-register 0x2102`
- Save configuration changes

### Disabling Password Recovery

```
no service password-recovery
```

- Hidden Cisco IOS command
- When configured, all access to ROMmon mode is disabled

Recovering a device after `no service password-recovery`

- Initiate break sequence within 5 seconds after image decompresses during the boot.
- Once confirming break, startup config is erased, password recovery procedure is enabled, and router boots with factory default configuration.
- *CAUTION:* If the router flash memory does not contain a valid Cisco IOS image because of corruption or deletion, the ROMmon xmodem command cannot be used to load a new flash image. To repair the router, an administrator must obtain a new Cisco IOS image on a flash SIMM or on a PCMCIA card. However, if an administrator has access to ROMmon they can restore an IOS file to flash memory using a TFTP server. Refer to Cisco.com for more information regarding backup flash images.

### Syslog Operation

Commands to set memory thresholds for when router will send notifications when available free memory falls below:

```
memory free low-watermark threshold io
memory free low-watermark processor
```

### Syslog Messages

```
service sequence-numbers
service timestamps
```

### System Logging Configuration

```
logging host {hostname | ip-address}
logging trap <level 0-7>                        ! optional
logging source-interface <int-type><int-number> ! optional
logging on
```

Example:

```
logging host 10.2.2.6
logging trap informational
logging source-interface g0/1
logging on
```

### Secure SNMPv3 Configuration

```
ip access-list standard <acl-name>
 permit <network-address>
 exit
snmp-server view <view-name> <oid-tree>
snmp-server group <group-name> v3 priv read <view-name> access [acl-number |acl-name]
snmp-server user <username> <group-name> v3 auth {md5 | sha} <auth-password> priv {des | 3des | aes {128 | 192 | 256}} <priv-password>
```

Example:

```
ip access-list standard PERMIT-ADMIN
 permit 10.10.10.1 0.0.0.255
 exit
snmp-server view SNMP-RO iso included
snmp-server group ADMIN v3 priv read SNMP-RO access PERMIT-ADMIN
snmp-server user Bob ADMIN v3 auth sha cisco12345 priv aes 128 cisco54321
```

Verify SNMPv3 Configuration:

```
show run | include snmp
show snmp user

```

Can use a SNMP MIB Browser to verify that a SNMP manager can send get requests to a router:
https://www.manageengine.com/products/mibbrowser-free-tool/

Keith Barker's demonstration of configuring and verifying SNMPv3:https://www.youtube.com/watch?v=XoMuYWol-7s

### Time and NTP Servers

Clock Setting:

```
show clock
clock set <...>
```

Configure device to be an authoritative NTP server:

```
ntp master [stratum-num]
```

Configure software clock to synchronize with an NTP time server:

```
ntp server {ip-address|hostname} [version <number>] [key <key-id>] [source <interface>] [prefer]
```

Configure device to receive NTP broadcast messages on the interface:

```
ntp broadcast client
```

Check status:

```
show ntp status
```

### NTP Authentication

Enable authentication:

```
ntp authentication
```

Define authentication keys:

```
ntp authentication-key <key-num> md5 <key-value>
```

Authenticates identity of system to which NTP will synchronize:

```
ntp trusted-key <key-number>
```

Verify NTP settings:

```
show ntp associations detail
show ntp associations detail | include <ip address of ntp server>
```

Example:

```
ntp authenticate
ntp authentication-key 1 md5 cisco123
ntp trusted-key 1
```

Update NTP Calendar:

```
ntp update-calendar
ntp calendar-valid      ! May be needed instead of above for some routers
```
