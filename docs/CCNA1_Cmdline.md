CCNA1 Command Line Reference
============================

## Break Keys

- Ctrl-C: Ends configuration mode -> PrivEXEC; aborts to command prompt in setup mode.
- Ctrl-Z: Ends configuration mode -> PrivEXEC
- Ctrl-Shift-6: Interrupts command midstream

- Ctrl-R: Redisplays line

## Modes

```
enable	              # Enters PrivEXEC
disable               # Exits PrivEXEC -> UserEXEC

configure terminal    # Enters Global Configuration Mode (GCM)
end                   # Exits into PrivExec
exit                  # Exits into current -> previous mode
```

# Basic Commands

## Configuration Status

```
show running-config   # Shows current configuration in RAM
show startup-config   # Shows configuration stored in NVRAM
```

## Save Configuration

```
copy running-config startup-config
cop r s               # Shortcut
```

## Change Hostname

```
hostname <device_host_name>
```

## Clock

```
show clock

clock set ?
clock set hh:mm:ss dd MON yyyy
```

## Configure Passwords

### Secure Privilege Exec Mode

```
enable password <password>    # Password in clear text
enable secret <password>      # Uses MD5 hash encryption; overrides password above
```

### Secure Console Line

```
line console 0		            # Enters first console interface
password <password>
login				                  # Enables userEXEC access
```

### Secure VTY (Terminal) Lines

```
line vty 0 15
password <password>
login
```

### Encrypt Passwords

```
service password-encryption		# Applies weak encryption
```

## Configure Banner MOTD

```
banner motd # <message> #

```

- Delimiter (# above) could be any character that is not in the message.

## Switch Virtual Interface Configuration

```
interface vlan 1
ip address <ip-address> <subnet-mask>
no shutdown
```

## Display Interface Information

```
show interfaces
show ip interface brief
show ip route

show interface serial 0/0/0
show interface GigabitEthernet 0/0

show interface s0/0/0         # shortcut
show interface g0/0           # shortcut
```

## Configure Router Interface (Lab 6.4.3.3)

```
interface gigabitethernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

## Configure an Interface Description

```
description <description>
```

# Configure IPv6 Addressing on the Router

## Enable router to forward IPv6 packets
```
ipv6 unicast-routing
```

## Configure IPv6 addressing

```
ipv6 address <ipv6 address>/<prefix>
ipv6 address <ipv6 address> link-local
no shutdown
```

# Basic Security Practices

```
service password-encryption
security password min-length <num chars>
login block-for <seconds> attempts <num attempts> within <seconds>
line vty 0 4
exec-timeout 10
end
```

## Configure SSH

- Requires hostname and domain name configured.

```
hostname <host_name>
ip domain-name <domain.com>
crypto key generate rsa
username <username> secret <password>
line vty 0 15
login local
transport input ssh
transport input telnet ssh        # Telnet and SSH
```

## Remove Username

```
no username <username>
```

# Miscellaneous

## Disable DNS lookup

- Prevents router from translating typos into an IP address as if they were hostnames.

```
no ip domain-lookup
```

## Shutdown Multiple Interfaces at a time

```
interface range f0/1-4, f0/7-24, g0/1-2
shutdown
end
```

## Show Commands

```
show arp
show flash
show ip route
show interfaces
show ip interface brief
show protocols
show users				# Show connected users
show version
show run | i username			# Shows list of usernames
```

## Debugging

```
debug ip icmp
undebug all
terminal monitor			# Show log messages through SSH
terminal no monitor			# Disable display of log messages through SSH
```

## Cisco Discovery Protocol CDP

```
show cdp neighbors
show cdp neighbors detail	# Shows IP addresses of neighbouring Cisco devices

no cdp run					# Disables CDP function - should be disabled for user-facing devices
```

### Backup Configuration using TFTP

- [Cisco Backup Config](http://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-software-releases-122-mainline/46741-backup-config.html)

```
copy startup-config tftp:
copy flash tftp:
```

## Flash

```
show flash
copy startup-config flash
```

## TFTP

```
copy startup-config tftp:
copy flash tftp:

```

## Connecting to SSH in Cisco Packet Tracer CMD Line

```
ssh -l <username> <target>
```

## Terminal Configuration Settings to Access Console
- Bits per second: 9600
- Data bits: 8
- Parity: None
- Stop bits: 1
- Flow control: None
