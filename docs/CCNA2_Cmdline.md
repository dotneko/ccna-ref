CCNA2 Command Line Reference
============================

## Enable IP on a Switch

```
interface vlan 1
ip address <xxx.xxx.xxx.xxx> <subnet mask>
no shutdown
exit
ip default-gateway <xxx.xxx.xxx.xxx>
```

# Router Configuration

### Configure Basic Router Settings

Always configure:

- Hostname
- Secure management access (privEXEC, userEXEC, remote access)
- Banner for legal notification of unauthorized access

### Configure an IPv4 Router Interface

- Good practice to configure `description` on each interface.
- Additional parameters might be required on certain interfaces - e.g. the serial interface connecting to the serial cable end labeled DCE must be confgured with the `clock rate` command.

```
interface serial 0/0/0
description Link to XX
ip address <ip_address> <subnet mask>
clock rate 128000
no shutdown
exit
```

### Configure an IPv6 Router Interface

```
interface GigabitEthernet 0/0
ipv6 address <ipv6_address>/<prefix-length>
ipv6 address <link-local address> link-local
no shutdown
exit
```

Note: An interface can generate its own IPv6 link-local address without having a global unicast address by using the `ipv6 enable` interface configuration command.

IPv6 interfaces typically have more than one IPv6 address.

- IPv6 device must have an IPv6 link-local address at a minimum.
- Most would also have an IPv6 global unicast address.
- IPv6 also supports ability for an interface to have multiple IPv6 global unicast addresses from the same subnet.

#### Statically create IPv6 global unicast/link-local addresses

Create a global unicast IPv6 address:

```
ipv6 address <ipv6-address>/<prefix-length>
```

Configure global unicast IPv6 address with an interface ID in the low-order 64 bits of the IPv6 address using the EUI-64 process.

```
ipv6 address <ipv6-address>/<prefix-length> eui-64
```

Configure a static link-local address on the interface that is used instaead of the link-local address that is automatically configured when the global unicast IPv6 address is assigned to the interface or enabled using the `ipv6 enable` interface command.

```
ipv6 address <ipv6-address>/<prefix-length> link-local
```

Configure IPv6 Unicast Routing

```
ipv6 unicast-routing
```

- Router begins sending ICMPv6 Router Advertisement messages out of the interface.
- This allows connected PCs to automatically configure an IPv6 address and set a default gateway without needing the services of a DHCPv6 server.

#### Enabling and Assigning a Loopback Address

```
interface loopback <number>
ip address <ip-address> <subnet-mask>
exit
```

### Verify Interface Settings

```
show ip interface brief
show ip route
show running-config interface <interface-id>
show interfaces
show ip interface
```

### Verify IPv6 Interface Settings

```
show ipv6 interface brief
```

- `up/up` output on the same line as the interface name indicates the Layer1/Layer2 state - same as the Status and Protocol columns in the equivalent IPv4 command.

```
show ipv6 interface gigabitethernet 0/0
show ipv6 route
```

- "C" next to route: directly connected.
- When the router interface is configured with a global unicast address and is in the “up/up” state, the IPv6 prefix and prefix length is added to the IPv6 routing table as a connected route.
- The IPv6 global unicast address configured on the interface is also installed in the routing table as a local route. The local route has a /128 prefix. Local routes are used by the routing table to efficiently process packets with the interface address of the router as the destination.
- The `ping` command for IPv6 is identical to the command used with IPv4 except that an IPv6 address is used. As shown in Figure 4, the ping command is used to verify Layer 3 connectivity between R1 and PC1.

### Filter Show Command Output

Configure number of lines to display during `show`

```
terminal length <number of lines>
terminal length 0     # No pause
```

Pipe after the `show` then `<filter parameter> <filter expression>`
Can be used in combination with any `show` command.

```
show running-config | section line vty
```

Filtering parameters:

- `section`: entire section that starts with expression
- `include`: all output lines that match expression
- `exclude`: excludes all output lines that match expression
- `begin`: all output lines from a certain point, starting with the line that matches the filtering expression.

Useful examples:

```
show ip interface brief | include up
show ip interface brief | exclude unassigned
show ip route | begin Gateway
```

## Command History

- Ctrl-P or Up-Arrow
- Ctrl-N or Down-Arrow

```
show history    # (PrivExec)
```

### Change size of history buffer

```
terminal history size <num>   # (UserExec)
```

## IPv6 Packets

- Instead of the ARP process, IPv6 address resolution uses ICMP v6 Neighbor Solicitation (NS) and Neighbor Advertisement messages.
- IPv6-to-MAC address mapping are kept in a table similar to the ARP cache, called the **neighbor cache**.
- More info on [IPv6 neighbor solicitation](https://keepingitclassless.net/2011/10/neighbor-solicitation-ipv6s-replacement-for-arp/)

```
show ipv6 neighbors
```

### Routing Table Sources

```
show ip route
show ipv6 route
```

# Configuring Static Routes

There are two common types of static routes in the routing table:

- Static route to a specific network
- Default static route

A static route can be configured to reach a specific remote network. IPv4 static routes are configured using the following command:

```
ip route <network-address> <subnet-mask> { next-hop-ip | exit-intf }
```

A static route is identified in the routing table with the code ‘S’.

A **default static route** is similar to a default gateway on a host. The default static route specifies the exit point to use when the routing table does not contain a path for the destination network. A default static route is useful when a router has only one exit point to another router, such as when the router connects to a central router or service provider.

To configure an IPv4 default static route, use the following command:

```
Router(config)# ip route 0.0.0.0 0.0.0.0 { exit-intf | next-hop-ip }
```

### IP Route Command

Requires following parameters:

- Network-address (destination); often referred to as the **prefix**
- Subnet-mask (or just mask); can be modified to summarize a group of networks.

Also one or both of the following:

- IP-address of the connecting router to use for forwarding; commonly referred to as the **next hop**.
- Exit-intf: the outgoing interface to use to forward the packet to the next hop.

The distance parameter is used to create a floating static route by setting an administrative distance that is higher than a dynamically learned route.

```
ip route <network-address> <subnet-mask> {ip-address | exit-intf}
```

### Next Hop Options

The next hop can be identified by an IP address, exit interface, or both. How the destination is specified creates one of the three following route types:

- Next-hop route - Only the next-hop IP address is specified
- Directly connected static route - Only the router exit interface is specified
- Fully specified static route - The next-hop IP address and exit interface are specified

```
show ip route | begin Gateway
show ip route | include C
```

### Configure an IPv4 Floating Static Route

- Given an example where R1 is attached to R2 (172.16.2.2) and R3 (10.10.10.2), the following would create a default static route to R2, and a floating static route to R3:

```
ip route 0.0.0.0 0.0.0.0 172.16.2.2     # Default route to R2
ip route 0.0.0.0 0.0.0.0 10.10.10.2 5   # Floating default route to R3
```

- Default route to R2 has no administrative distance specified, so would default to 1. This is the preferred route..
- Floating default route to R3 has administrative distance 5. Since this value is greater that of the default route, this route "floats" - it is not present in the routing table unless the preferred route fails.

### Verify a Static Route

```
show ip route
show ip route static                  # Displays contents of static routes
show ip route static | begin Gateway
show ip route <network>

show running-config | section ip route
traceroute <ip-address>
```

## IPv6 Routes

Static routes for IPv6 are configured using the ipv6 route global configuration command.

```
ipv6 route <ipv6-prefix/prefixlength> {ipv6-address | exit-intf}
```

### Displaying and Testing IPv6 Routes

```
show ipv6 route
show ipv6 route static
show ipv6 route <network>
show running-config | section ipv6 route

ping ipv6 <ipv6-address>
traceroute <
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

# Configuring Dynamic Routing

### Access RIP configuration mode

```
router rip
no router rip
```
- Prompt will show up as `Router(config-router)#`
- When enabling RIP, the default version is RIPv1.
- To disable and eliminate RIP, use the `no router rip` global configuration command. This command stops the RIP process and erases all existing RIP configurations.

### Advertise Networks

```
network <network-address>
```

### Verify RIP Routing

```
show ip protocols
```

### Enable and Verify RIPv2

By *default, when a RIP process is configured on a Cisco router, it is running RIPv1*, as shown in Figure 1. However, even though the router only sends RIPv1 messages, it can interpret both RIPv1 and RIPv2 messages. A *RIPv1 router ignores the RIPv2 fields in the route entry*.

Use the `version 2` router configuration mode command to enable RIPv2, as shown in Figure 2. Notice how the `show ip protocols` command verifies that R2 is now configured to send and receive version 2 messages only. The RIP process now includes the subnet mask in all updates, making *RIPv2 a classless routing protocol*.

Note: Configuring `version 1` enables RIPv1 only, while configuring `no version` returns the router to the default setting of sending version 1 updates but listening for version 1 and version 2 updates.

### Disable Auto Summarization

```
no auto-summary
```

### Propogate a Default Route

To propagate a default route in RIP, the edge router must be configured with:

- A default static route using the `ip route 0.0.0.0 0.0.0.0` command.
- The `default-information originate` router configuration command. This instructs R1 to originate default information, by propagating the static default route in RIP updates.


