### Mitigate CDP/LLDP reconnaissance attacks

Disable CDP/LLDP globally:

```
no cdp run
no lldp run
```

Disable CDP/LLDP on a port:

```
interface <intf-id>
  no cdp enable
  no lldp transmit
  no lldp receive
```

### Mitigate Telnet attacks

- Use SSH and strong passwords
- Limit access to vty lines with ACLs
- Authenticate and authorize admin access with AAA (TACACS+ or RADIUS)

### Mitigate MAC Address Table flooding attacks

Show MAC address table:

```
show mac address-table
```

Configure port security to mitigate MAC address table overflow attacks.

```
interface <intf-id>
  switchport mode access
  switchport port-security
  switchport port-security maximum 5
  switchport port-security mac-address sticky
  switchport port-security violation [ protect|restrict|shutdown ]
```

### Mitigate VLAN Attacks


Ways to mitigate VLAN attacks:

- Explicitly configure access links
- Explicitly disable auto trunking
- Manually enable trunk links
- Disable unused ports, make them access ports, and assign them to a black hole VLAN
- Change the default native VLAN
- Implement port security

Prevent basic VLAN attacks:

```
switchport mode access  ! For non-trunking ports
switchport mode trunk   ! Manually enable trunk link
switchport nonegotiate  ! Disable DTP (autotrunking) negotations
switchport tunk native vlan <unused vlan num>

interface range <intf-range>
  switchport mode access
  switchport access vlan <unused VLAN number>
  shutdown              ! Disable unused ports

```

### DHCP Attacks

- DHCP starvation is often used before a DHCP spoofing attack to deny service to the legitimate DHCP server.
- Configure DHCP snooping and port security on the switch to mitigate DHCP attacks.

### Strategies to help secure Layer 2 of a network:

- Always use secure variants of these protocols such as SSH, SCP, SSL, SNMPv3, and SFTP.
- Always use strong passwords and change them often.
- Enable CDP on select ports only.
- Secure Telnet access.
- Use a dedicated management VLAN where nothing but management traffic resides.
- Use ACLs to filter unwanted access.

### Secure Admin Access using AAA

See [CCNA Security:AAA](../CCNAS/S03_Basic_AAA_Configuration.md)

# SNMP

### SNMPv2 Configuration

```
snmp-server community <string> ro|rw [SNMP_ACL]
snmp-server location <text, e.g. NOC_SNMP_MANAGER>
snmp-server contact John Doe
snmp-server host <host-id> [version {1| 2c | 3 \
          [auth | noauth | priv]}] <community-string>
```

To restrict SNMP access to NMS hosts, define ACL and reference it as above:

```
ip access-list standard SNMP_ACL
  permit 192.168.1.3
```

To enable traps on an SNMP agent (none set by default):

```
snmp-server enable traps <notification-types>
```

### Verify SNMP

```
show snmp
show snmp community
```

### Best Practices

- Choose community strings careful; SNMPv1 and SNMPv2c rely on SNMP community strings in plaintext to authenticate access to MIB objects.
- If SNMP is used only to monitor devices, use read-only communities.
- Ensure that SNMP messages do not spread beyond management consoles; use ACLs.
- Use SNMPv3 as it provides security authentication and encryption.

### SNMPv3 Configuration

Create a new SNMP group and add a new user to it:

```
snmp-server group <groupname> {v1|v2c|v3 {auth|noauth|priv}}
snmp-server user <username> <groupname> v3 [encrypted] \
            [auth {md5|sha} <auth-password>] \
            [priv {des|3des|aes {128|192|256}} <priv-password>]
```

Example:

```
ip access-list standard PERMIT-ADMIN
  permit 192.168.1.0 0.0.0.255
  exit

snmp-server view SNMP-RO iso included
snmp-server group ADMIN v3 priv read SNMP-RO access PERMIT-ADMIN
snmp-server user BOB ADMIN v3 auth sha cisco12345 priv aes 128 cisco54321
```

# Switched Port Analyzer (SPAN) Configuration

- The destination port cannot be a source port, and the source port cannot be a destination port.
- The number of destination ports is platform-dependent. Some platforms allow for more than one destination port.
- The destination port is no longer a normal switch port. Only monitored traffic passes through that port.

### Local SPAN Configuration

```
monitor session <number> source [interface <interface>|vlan <vlan-id>]
monitor session <number> destination [interface <interface>|vlan <vlan-id>]
```

Verify:

```
show monitor
```
