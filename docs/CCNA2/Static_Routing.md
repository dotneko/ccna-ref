# Configuring Static Routes

There are two common types of static routes in the routing table:

- Static route to a specific network
- Default static route

# IPv4 Routing

```
ip route <network-address> <subnet-mask> { next-hop-ip | exit-intf }
```
- Next-hop-ip: IP-address of the connecting router to use for forwarding.
- Exit-intf: the outgoing interface to use to forward the packet to the next hop.


### Configure a IPv4 Default Static Route

```
ip route 0.0.0.0 0.0.0.0 { exit-intf | next-hop-ip }
```
The distance parameter is used to create a floating static route by setting an administrative distance that is higher than a dynamically learned route.

### Configure an IPv4 Floating Static Route

- Given an example where R1 is attached to R2 (172.16.2.2) and R3 (10.10.10.2), the following would create a default static route to R2, and a floating static route to R3:

```
ip route 0.0.0.0 0.0.0.0 172.16.2.2     # Default route to R2
ip route 0.0.0.0 0.0.0.0 10.10.10.2 5   # Floating default route to R3
```

- Default route to R2 has no administrative distance specified, so would default to 1. This is the preferred route..
- Floating default route to R3 has administrative distance 5. Since this value is greater that of the default route, this route "floats" - it is not present in the routing table unless the preferred route fails.

### Default Administrative Distances

```
Connected                   0
Static                      1
EIGRP summary route         5
External BGP                20
Internal EIGRP              90
IGRP                        100
OSPF                        110
IS-IS                       115
RIP                         120
External EIGRP              170
Internal BGP                200
```

### Verify a Static Route

```
show ip route
show ip route static                  # Displays contents of static routes
show ip route static | begin Gateway
show ip route <network>

show running-config | section ip route
traceroute <ip-address>
```

# IPv6 Routing

```
ipv6 route <ipv6-prefix>/<prefixlength> {ipv6-address | exit-intf}
```

### Displaying and Testing IPv6 Routes

```
show ipv6 route
show ipv6 route static
show ipv6 route <network>
show running-config | section ipv6 route

ping ipv6 <ipv6-address>
traceroute
```

### Configure a Directly Connected Static IPv6 Route

```
ipv6 route <ipv6-address>/<network-prefix> <interface>
ipv6 route 2001:db8:acad:2::/64 s0/0/0
```

### Configure a Fully Specified Static IPv6 Route

```
ipv6 route <ipv6-address>/<network-prefix> <interface> <next-hop ipv6-address>
ipv6 route 2001:db8:acad:2::/64 s0/0/0 fe80::2
```

### Configure a Default IPv6 Static Route

```
ipv6 route ::/0 {ipv6-address | exit-intf}
ipv6 route ::/ 2001:db8:acad:4::2
```

### Configure an IPv6 Floating Static Route

```
ipv6 route ::/0 <ipv6-address> <administrative-distance>
ipv6 route ::/0 2001:db8:aad:4::2
ipv6 route ::/0 2001:db8:aad:6::2 5
```

# Common IOS Troubleshooting Commands

```
ping
ping <ip-address> source <source_ip>
traceroute <ip-address>
show ip route | begin Gateway
show ip interface brief
show cdp neighbors detail
```


