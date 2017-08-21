### Troubleshooting Commands

```
show processes cpu
show memory
show interfaces
show interfaces <intf-id>

show version
show ip interface [brief]
show ipv6 interface [brief]
show interfaces [<intf-id>]
show arp
show ipv6 neighbors
show running-config
show port
show vlan
show tech-supprot
show ip cache flow

[no] debug ?
show protocols
```

### IP SLA Configuration

```
ip sla <operation-num>
  icmp-echo { <dest-ip-address> | <dest-hostname> } \
            [ source-ip <source-ip-addr> | <hostname> } | \
            source-interface <intf-id>
ip sla schedule <operation-num> [ life { forever | <seconds> }] \
            [ start-time { <hh>:<mm>:[:<ss>] { <month>] \
            | pending | now | after <hh:mm:ss> ] \
            [ ageout <seconds> ] [ recurring ]

no ip sla <operation-num>
```

Verify/Check statistics:

```
show ip sla application
show ip sla configuration [<operation-num>]
show ip sla statistics [<operation-num>]
```

Example:

```
conf t
ip sla 1
  icmp-echo 192.168.1.5   ! Monitor interface at 192.168.1.5
    frequency 30
    exit
ip sla schedule 1 start-time now life forever
```

### Windows Addressing and Mappings

```
arp
arp -d      ! Clears cache for repopulation
```

Check IPv6 Neighbor Table

```
netsh interface ipv6 show neighbor
```

### Linux/Unix Addressing and Mappings

```
ifconfig

ip neigh show
```
