# Switches: Basic Configuration

### Enable IP on a Switch

```
interface vlan 1
ip address <xxx.xxx.xxx.xxx> <subnet mask>
no shutdown
exit
ip default-gateway <xxx.xxx.xxx.xxx>
```

### Configuring Switch Management Interface

```
interface vlan 99
ip address <network-address> <subnet-mask>
no shutdown
end

ip default-gateway <ip-address>                 # (GCM)
```

Note that above vlan99 will not be up until a new VLAN is created:

### Create a new VLAN

```
vlan <vlan_id>
name <vlan_name>
exit
interface <interface-id>
switchport mode access
switchport access vlan <vlan_id>
end

# E.g.
vlan 99
name new99
```

### Configure Duplex and Speed, Auto-MDX

```
duplex full             # (Interface config)
speed 100
```

Auto-MDX
```
duplex auto
speed auto
mdix auto               # Requires speed/duplex set to auto

show controllers ethernet-controller int   # Show Auto-MDIX setting

show controllers ethernet-controller fa 0/1 phy | include Auto-MDIX
```

- Enabled by default on Catalyst 2960 and 3560 switches.
- Not available on older Catalyst 2950

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

Configure a static link-local address on the interface that is used instead of the link-local address that is automatically configured when the global unicast IPv6 address is assigned to the interface or enabled using the `ipv6 enable` interface command.

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

### IPv6 Packets

- Instead of the ARP process, IPv6 address resolution uses ICMP v6 Neighbor Solicitation (NS) and Neighbor Advertisement messages.
- IPv6-to-MAC address mapping are kept in a table similar to the ARP cache, called the **neighbor cache**.
- More info on [IPv6 neighbor solicitation](https://keepingitclassless.net/2011/10/neighbor-solicitation-ipv6s-replacement-for-arp/)

```
show ipv6 neighbors
```


