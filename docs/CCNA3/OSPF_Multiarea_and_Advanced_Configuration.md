### Multiarea OSPFv2

ABR Example through interface advertisement:

```
router ospf 10
  router-id 1.1.1.1
  network 10.1.1.1 0.0.0.0 area 1
  network 10.1.2.1 0.0.0.0 area 1
  network 192.168.10.1 0.0.0.0 area 0
  end
```

### Multiarea OSPFv3 (IPv6)

Example:

```
ipv6 router ospf 10
  router-id 1.1.1.1
  exit
  
interface GigabitEthernet 0/0
  ipv6 ospf 10 area 1
  
interface Serial 0/0/0
  ipv6 ospf 10 area 0
  end
```

### Verifying Multiarea OSPFv2

The same verification commands used to verify single-area OSPFv2 also can be used to verify the multiarea OSPF topology:

```
show ip ospf neighbor
show ip ospf
show ip ospf interface
```

Commands that verify specific multiarea OSPFv2 information include:

```
show ip protocols
show ip ospf interface brief
show ip route ospf
show ip ospf database
```

Note: For the equivalent OSPFv3 command, simply substitute `ip` with `ipv6`.

### Using a Loopback Interface for a Simulated OSPF Network (Lab 9.2.2.9)

```
ipv6 router ospf 1
 router-id 1.1.1.1
 
interface lo0
 ipv6 ospf 1 area 1
 ipv6 ospf network point-to-point
```

# OSPF in MultiAccess Networks

### Verify OSPF Router Roles and Neighbor Adjacencies

```
show ip ospf interface <intf-id>
show ip ospf neighbor
```

### OSPF Priority Configuration

```
ip ospf priority <value>    ! OSPFv2 interface command
ipv6 ospf priority <value>  ! OSPFv3 interface command
```

The value can be:

- `0` - Does not become a DR or BDR.
- `1–255` - The higher the priority value, the more likely the router becomes the DR or BDR on the interface.

### Propagating a Default Static Route in OSPFv2

To propagate a default route, the edge router must be configured with:

- Default static route: `ip route 0.0.0.0 0.0.0.0 {<ip-address> | <exit-intf>}`
- Propagate in SPF updates: `default-information originate`

Example:

```
ip router 0.0.0.0 0.0.0.0 209.165.200.226

router ospf 10
  default-information originate
  end
```

### Propagating a Default Static Route in OSPFv3

To propagate a default route, the edge router must be configured with:

- Default static route: `ipv6 route ::/0 {<ipv6-address> | <exit-intf>}`
- Propagate route: `default-information originate` 

Example:

```
ipv6 route ::/0 201:db8:feed:1::2

ipv6 router ospf 10
  default-information originate
  end
```

### Modify OSPFv2 Intervals

Note: The *default Hello and Dead intervals are based on best practices and should only be altered in rare situations*.

OSPFv2 Hello and Dead intervals can be modified manually using the following interface configuration mode commands:

```
ip ospf hello-interval <seconds>
ip ospf dead-interval <seconds>
```

Use the `no ip ospf hello-interval` and `no ip ospf dead-interval` commands to reset the intervals to their default.

Immediately after changing the Hello interval, the **Cisco IOS automatically modifies the Dead interval to four times the Hello interval**. However, it is *always good practice to explicitly modify the timer instead of relying on an automatic IOS feature so that modifications are documented in the configuration*.

### Modify OSPFv3 Intervals (IPv6)

OSPFv3 Hello and Dead intervals can be modified manually using the following interface configuration mode commands:

```
ipv6 ospf hello-interval <seconds>
ipv6 ospf dead-interval <seconds>
```

Note: Use the `no ipv6 ospf hello-interval` and `no ipv6 ospf dead-interval` commands to reset the intervals to their default.

### Fix Mismatched MTU Sizes

If two connecting routers had mismatched MTU values, they would still attempt to form an adjacency but they would not exchange their LSDBs and the neighbor relationship would fail.

```
ip mtu <size>
ipv6 mtu <size>
```

### Troubleshooting Commands

```
show ip protocols
show ip ospf neighbor
show ip ospf interface
show ip ospf
show ip route ospf
clear ip ospf [<process-id>] process
```

Troubleshooting OSPFv3 is similar to OSPFv2. The following commands are the equivalent commands used with OSPFv3: 

```
show ipv6 protocols
show ipv6 ospf neighbor
show ipv6 ospf interface
show ipv6 ospf
show ipv6 route ospf
clear ipv6 ospf [<process-id>] process
```

# Additional Commands

### Configuring Interarea Route Summarization

Example - given 4 loopback interfaces with sequential network addresses

a.	List the network addresses for the loopback interfaces and identify the hextet section where the addresses differ.

```
2001:DB8:ACAD:0000::1/64
2001:DB8:ACAD:0001::1/64
2001:DB8:ACAD:0002::1/64
2001:DB8:ACAD:0003::1/64
```

b.	Convert the differing section from hex to binary.

```
2001:DB8:ACAD: 0000 0000 0000 0000::1/64
2001:DB8:ACAD: 0000 0000 0000 0001::1/64
2001:DB8:ACAD: 0000 0000 0000 0010::1/64
2001:DB8:ACAD: 0000 0000 0000 0011::1/64
```

c.	Count the number of leftmost matching bits to determine the prefix for the summary route.

```
2001:DB8:ACAD: 0000 0000 0000 0000::1/64
2001:DB8:ACAD: 0000 0000 0000 0001::1/64
2001:DB8:ACAD: 0000 0000 0000 0010::1/64
2001:DB8:ACAD: 0000 0000 0000 0011::1/64
```

To summarize configure interarea route summarization:

```
ipv6 router ospf 1
 area 1 range 2001:db8:acad::/62
```

### OSPFv2 Authentication with MD5

Can see [Sample Configuration from Cisco.com](http://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13697-25.html) for further details.

```
! Router Interface configuration:
interface <intf-id>
  ip ospf message-digest-key <key-num> md5 <password>
  ip ospf authentication message-digest
  exit

! To use authentication for all interfaces in an area:
router ospf <process-id>
  area <area-id> authentication message-digest
```
