### Display and Verify DHCPv4

```
show running-config | section dhcp      # Doesn't work in packet tracer
show ip dhcp binding
show ip dhcp server statistics
```

### DHCPv4: Basic Configuration

Steps:

1. Exclude IPv4 Addresses: e.g. those assigned to devices that require static addresses - routers, servers, printers, other manually configured devices.

```
ip dhcp excluded-address <single-ip>
ip dhcp excluded-address <low-address> <high-address>
```

2. Configuring a DHCPv4 Pool that will be used for assignment

```
ip dhcp pool <pool-name>        # Enters DHCPv4 config mode
```

3. Configuring Specific Tasks

Required:

```
network <network-num> [ mask | /prefix-length ]
default-router <address [ address2 ... address8 ]
```

Optional:

```
dns-server <address> [ address2 .. address8 ]
domain-name <name>
lease ( days [hours] [minutes] | infinite )
netbios-name-server <address> [ address2 ... address8 ]
```

Example:

```
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.254
ip dhcp pool LAN-POOL-1
  network 192.168.10.0 255.255.255.0
  default-router 192.168.10.1
  dns-server 192.168.11.5
  domain-name example.com
  exit
```

### Release/Renew IP Address on Windows

```
ipconfig /release
ipconfig /renew         # PC will broadcast a DHCPDISCOVER message
```

### DHCPv4 Relay Configuration

```
interface <interface-id>
  ip helper-address <DHCP-IPv4-server-address>
  exit
```

Example:

- Router1:
    - G0/0 connected to network1 192.168.10.0/24 with PC1
    - G0/1 connected to network2 192.168.20.0/24 with DHCP server at 192.168.20.5
- Router1 will need to setup a DHCPv4 relay for PC1 to be able to use DHCP on network2.

```
interface g0/0
  ip helper-address 192.168.20.5
  end
  
show ip interface   # Verify configuration
```

### DHCPv4: Troubleshooting

1. Resolve IPv4 address conflicts
2. Verify physical connectivity
3. Test with a static IPv4 address
4. Verify switch port configuration
5. Test from the same subnet or VLAN

```
show ip dhcp conflict
```

### Verify Router DHCPv4 Configuration

If the IPv4 helper address is not configured properly (for when DHCPv4 server is located on a separate LAN from the client), client DHCPv4 requests are not forwarded to the DHCPv4 server.

```
show running-config | section interface <interface-id>
show running-config | include no service dhcp
```

### DHCPv4 on Routers: Debugging

Verify that the router is receiving the DHCPv4 request from the client:

- The DHCPv4 process fails if the router is not receiving requests from the client.
- This involves configuring an ACL for debugging output.

```
conf t
  access-list 100 permit udp any any eq 78
  access-list 100 permit udp any any eq 68
  end
  
debug ip packet 100
debug ip dhcp server events
```

### DHCPv6 Router Configuration

IPv6 routing must b enabled before a router can send RA messages:

```
ipv6 unicast-routing
```

RA messages are configured on an individual interface of a router. To re-enable an interface for SLAAC that might have been set to another option, the M and O flags need to be reset to their initial values of 0. This is done using the following interface configuration mode commands:

```
Router(config-if)# no ipv6 nd managed-config-flag 
Router(config-if)# no ipv6 nd other-config-flag 
```

### Stateless DHCPv6

For stateless DHCPv6, the O flag is set to 1 and the M flag is left at the default setting of 0. The O flag value of 1 is used to inform the client that additional configuration information is available from a stateless DHCPv6 server.

To modify the RA message sent on the interface of a router to indicate stateless DHCPv6, use the following command:

```
Router(config-if)# ipv6 nd other-config-flag 
```

### Stateful DHCPv6

The M flag indicates whether or not to use stateful DHCPv6. The O flag is not involved. The following command is used to change the M flag from 0 to 1 to signify stateful DHCPv6:

```
Router(config-if)# ipv6 nd managed-config-flag 
```

