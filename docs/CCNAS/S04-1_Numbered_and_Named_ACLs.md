### Configuring Numbered and Named ACLs

##### Standard ACL Syntax

```
access-list <acl-#> {permit|deny|remark} source-addr [source-wildcard] [log]
```

##### Extended ACL Syntax

```
access-list <acl-#> {permit|deny|remark} <protocol> <source-addr> [source-wildcard] <dest-addr> [dest-wildcard][operator port] [established]
```

Examples:

- On R2 S0/0/0 that connects to web server at 10.1.3.8, allow only users on 10.1.1.0/24 network to have HTTP access

```
access-list 101 permit tcp 10.1.1.0 0.0.0.255 host 10.1.3.8 eq 80
```

- On R2 G0/1 that connects to 10.1.2.0/24, block host 10.1.2.9 from having FTP access to 10.1.1.0/24 network (G0/0)

```
access-list 122 deny tcp host 10.1.2.9 10.1.1.0 0.0.0.255 eq 21
```

- On R1 G0/0 (inbound) that connects to 10.1.3.0/24, allow only host 10.1.3.8 to reach destinations beyond that network (S0/0/0)

```
access-list 150 permit ip host 10.1.3.8 any
```

##### Named ACL Syntax:

```
! Name ACL
ip access-list [standard|extended] <ACL name>

! Configure ACEs
[permit|deny|remark] {source [source-wildcard] |any}
[permit|deny|remark] <protocol> <source-addr [source-wildcard]
```

### Applying and ACL

After creating an ACL, the administrator can apply it in a number of different ways.

Syntax for Applying an ACL to an Interface:

```
interface <interface-id>
  ip access-group {acl-#|name} {in|out}
```

Syntax for Applying an ACL to the VTY Lines:

```
line vty 0 4
  access-class {acl-#|name} {in|out}
```

Named Standard ACL Example:

```
ip access-list standard NO_ACCESS
  deny host 192.168.11.10
  permit any
  exit
interface g0/0
  ip access-group NO_ACCESS out
```

Named Extended ACl Example:

- The SURFING ACL is applied to inbound traffic and the BROWSING ACL is applied to outbound traffic.

```
ip access-list extended SURFING
  permit tcp 192.168.10.0 0.0.0.255 any eq 80
  permit tcp 192.168.10.0 0.0.0.255 any eq 443
  exit
ip access-list extended BROWSING
  permit tcp any 192.168.10.0 0.0.0.255 established
  exit
interface g0/0
  ip access-group SURFING in
  ip access-group BROWSING out
```

Named ACL on VTY Lines with Logging:

```
ip access-list standard VTY_ACCESS
  permit 192.168.10.10 log
  deny any
  exit
line vty 0 4
  access-class VTY_ACCESS in
  end

! Matches to VTY access on subsequent logging can be viewed by:

show access-lists
```

- Logging has been enabled with the `log` parameter. Log messages are generated on the first packet match and then at five minute intervals after that first packet match. You can use the `show access-list` command to see how many packets have matched a statement.

### ACL Configuration Guidelines

- Create an ACL globally and then apply it.
- Ensure the last statement is an implicit `deny any` or `deny any any`
- Remember that statement order is important because ACLs are processed top-down. As soon as a statement is matched the ACL is exited.
- Ensure that the most specific statements are at the top of the list.
- Remember that only one ACL is allowed per interface, per protocol, per direction.
- Remember that new statements for an existing ACL are added to the bottom of the ACL by default.
- Remember that router generated packets are not filtered by outbound ACLs.
- Place standard ACLs as close to the destination as possible.
- Place extended ACLs as close to the source as possible.

### Antispoofing with ACLs

Where the interface is attached to the Internet, it should never accept inbound packets from the following addresses:

- All zeros addresses
- Broadcast addresses
- Local host addresses (127.0.0.0/8)
- Reserved private addresses (RFC 1918)
- IP multicast address range (224.0.0.0/4)

__Example:__ The 192.168.1.0/24 network is attached to the R1 G0/0 interface, while R1 S0/0/0 connects to the Internet The interface G0/0 should only allow inbound packets with a source address from that network. The ACL for G0/0 shown in the figure will only permit inbound packets from the 192.168.1.0/24 network. All others will be discarded.

```
! Inbound s0/0/0

access-list 150 deny ip host 0.0.0.0 any
access-list 150 deny ip 10.0.0.0 0.255.255.255 any
access-list 150 deny ip 127.0.0.0 0.255.255.255 any
access-list 150 deny ip 172.16.0.0 0.15.255.255 any
access-list 150 deny ip 192.168.0.0 0.0.255.255 any
access-list 150 deny ip 224.0.0.0 15.255.255.255 any
access-list 150 deny ip host 255.255.255.255 any

! Inbound g0/0

access-list 105 permit ip 192.168.1.0 0.0.0.255 any
```

### Permitting Necessary Traffic through a Firewall

An effective strategy for mitigating attacks is to **explicitly permit only certain types of traffic through a firewall**.

__Example:__ sample topology with possible ACL configurations to permit specific services on the Serial 0/0/0 interface.

- R1 S0/0/0: Internet via 10.0.1.1
- R1 G0/1: PC1
- R1 G0/0: Server 192.16.8.20.2/24 for DNS, SMTP, FTP
- Remote Access from 200.5.5.5/24 via Internet

```
! Inbound on S0/0/0
access-list 180 permit udp any host 192.168.20.2 eq domain
access-list 180 permit tcp any host 192.168.20.2 eq smtp
access-list 180 permit tcp any host 192.168.20.2 eq ftp
access-list 180 permit tcp host 200.5.5.5 host 10.1.1 eq 22
access-list 180 permit udp host 200.5.5.5 host 10.0.1.1 eq syslog
access-list 180 permit udp host 200.5.5.5 host 10.0.1.1. eq snmptrap
```

### Mitigating ICMP Abuse

Hackers can use *Internet Control Message Protocol (ICMP) echo packets (pings) to discover subnets and hosts on a protected network and to generate DoS flood attacks*. Hackers can use ICMP redirect messages to alter host routing tables. **Both ICMP echo and redirect messages should be blocked inbound by the router**.

Several ICMP messages are recommended for proper network operation and should be allowed into the internal network:

- **Echo reply** - Allows users to ping external hosts.
- **Source quench** - Requests that the sender decrease the traffic rate of messages.
- **Unreachable** - Generated for packets that are administratively denied by an ACL.

Several ICMP messages are required for proper network operation and should be allowed to exit the network:

- **Echo** - Allows users to ping external hosts.
Parameter problem - Informs the host of packet header problems.
- **Packet too big** – Enables packet maximum transmission unit (MTU) discovery.
- **Source quench** - Throttles down traffic when necessary.

As a rule, block all other ICMP message types outbound.

ACLs are used to block IP address spoofing, selectively permit specific services through a firewall, and to allow only required ICMP messages.

__Example:__ The figure shows a sample topology and possible ACL configurations to permit specific ICMP services on the G0/0 and S0/0/0 interfaces.

- R1 S0/0/0: Internet to Remote 209.16.201.3/24
- R1 G0/0: PC 192.168.1.2/24

```
! Inbound on S0/0

access-list 112 permit icmp any any echo-reply
access-list 112 permit icmp any any source-quench
access-list 112 deny icmp any any
access-list 112 pemit ip any any

! Inbound on G0/0
access-list 114 permit icmp 192.168.1.0 0.0.0.255 any echo
access-list 114 permit icmp 192.168.1.0 0.0.0.255 any parameter-problem
access-list 114 permit icmp 192.168.1.0 0.0.0.255 any packet-too-big
access-list 114 permit icmp 192.168.1.0 0.0.0.255 any source-quench
access-list 114 deny icmp any any
```

### Mitigating SNMP Exploits

Management protocols, such as SNMP, are useful for remote monitoring and management of networked devices. However, they can still be exploited. __If SNMP is necessary__, *exploitation of SNMP vulnerabilities can be mitigated by applying interface ACLs to filter SNMP packets from non-authorized systems*. An exploit may still be possible if the SNMP packet is sourced from an address that has been spoofed and is permitted by the ACL.

These security measures are helpful, but the **most effective means of exploitation prevention is to disable the SNMP server on IOS devices for which it is not required**. As shown in the figure, use the command `no snmp-server` to disable SNMP processing on the Cisco IOS devices.

```
no snmp-server
```

# 
