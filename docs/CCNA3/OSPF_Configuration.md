### Useful commands related to OSPF:

```
show ip ospf neighbor   ! Adjacency database
show ip ospf database   ! Link-state database
show ip route           ! Forwarding database
show ip protocols       ! Verify OSPF and router ID
```

Basic OSPFv2 Configuration:

```
router ospf <process-id>
  router-id <rid>
```

### Router ID Determination

1. The router ID is **explicitly configured** using the OSPF `router-id <rid>` router configuration mode command. The rid value is any 32-bit value expressed as an IPv4 address. This is the *recommended method to assign a router ID*.
2. If the router ID is not explicitly configured, the router chooses the **highest IPv4 address of any of configured loopback interfaces**. This is the next best alternative to assigning a router ID.
3. If no loopback interfaces are configured, then the router chooses the **highest active IPv4 address of any of its physical interfaces**. This is the least recommended method because it makes it more difficult for administrators to distinguish between specific routers.

### Modifying a Router ID

Clearing the OSPF process to reset the router ID:

```
! Router ID already exists, modify to 1.1.1.1
router ospf 10
  router-id 1.1.1.1

! Clear process to enable new router ID
clear ip ospf process
```

### Using a Loopback Interface as the Router ID

A router ID can also be assigned using a loopback interface.

The IPv4 address of the loopback interface should be **configured using a 32-bit subnet mask** (255.255.255.255). This effectively creates a host route. A *32-bit host route does not get advertised* as a route to other OSPF routers.

```
interface loopback 0
  ip address 1.1.1.1 255.255.255.255
```

### Enabling OSPF on Interfaces

```
network <network-address> <wildcard-mask> area <area-id>
```

Example:

```
router ospf 10
  network 172.16.1.0 0.0.0.255 area 0
  network 172.16.3.0 0.0.0.3 area 0
  network 192.168.10.4 0.0.0.3 area 0
```

**Using the quad 0 wildcard mask to specify the interface IPv4 address**

As an alternative, OSPFv2 can be enabled using the `network <intf-ip-address> 0.0.0.0 area <area-id>` router configuration mode command.

Example:

```
router ospf 10
  network 172.16.1.1 0.0.0.0 area 0
  network 172.16.3.1 0.0.0.0 area 0
  network 192.168.10.5 0.0.0.0 area 0
```

### Passive Interface Configuration

```
router ospf 10
  passive-interface G0/0
```

Verify with: `show ip protocols`

- Note: OSPFv2 and OSPFv3 both support the `passive-interface` command.
- As an alternative, all interfaces can be made passive using the `passive-interface default` command. Interfaces that should not be passive can be re-enabled using the `no passive-interface` command.

### Adjusting Reference Bandwidth

**OSPF uses a reference bandwidth of 100 Mb/s for any links that are equal to or faster than a fast Ethernet connection**. Therefore, the cost assigned to a fast Ethernet interface with an interface bandwidth of 100 Mb/s would equal 1.

**To assist OSPF in making the correct path determination, the reference bandwidth must be changed to a higher value to accommodate networks with links faster than 100 Mb/s**.

```
auto-cost reference-bandwidth <value; default = 100>

auto-cost reference-bandwidth 1000      ! Gigabit Ethernet
auto-cost reference-bandwidth 10000     ! 10 Gigabit Ethernet Links
```

Check/Verify current OSPFv2 cost:

```
show ip ospf interface <intf-id>
```

### Adjusting the Interface Bandwidth

- Check interface bandwidth with `show interfaces`
- Check clock rate with `show controllers`
- If the bandwidth setting is not accurate, the interface default interface bandwidth of `1,544 kb/s` may need to be adjusted.


```
show interfaces serial 0/0/1 | include BW
show ip ospf interface serial 0/0/1
```

Adjust bandwidth:

```
interface <intf-id>
  bandwidth <kilobits>
```

Example: Given a clock rate of 128000 bits on a serial interface, adjust bandwidth:

```
interface s0/0/0
  bandwidth 128
```

### Manually Setting the OSPF Cost

```
ip ospf cost <value>
```

An advantage of configuring a cost over setting the interface bandwidth is that the *router does not have to calculate the metric when the cost is manually configured*. In contrast, when the interface bandwidth is configured, the router must calculate the OSPF cost based on the bandwidth. The `ip ospf cost` command is useful in multi-vendor environments where non-Cisco routers may use a metric other than bandwidth to calculate the OSPFv2 costs.

### Verify OSPF Settings

```
show ip ospf neighbor
show ip protocols       ! Protocol Settings
show ip ospf            ! Process Information
show ip ospf interface
show ip ospf interface brief
```

# OSPFv3 (IPv6)

### Steps to Configure OSPFv3

1. Enable IPv6 unicast routing: `ipv6 unicast-routing`.
2. (Optional) Configure link-local addresses.
3. Enter OSPFv3 router configuration mode by `ipv6 router ospf <process-id>`, then configure a 32-bit router ID using the `router-id <rid>` command.
4. Configure optional routing specifics such as adjusting the reference bandwidth.
5. (Optional) Configure OSPFv3 interface specific settings. For example, adjust the interface bandwidth.
6. Enable IPv6 routing by using the `ipv6 ospf area` command.

### Changing the OSPF Router ID

**After an OSPFv3 router establishes a router ID, that router ID cannot be changed until the router is reloaded or the OSPFv3 process is cleared**.

```
clear ipv6 ospf process
```

Doing this forces OSPF on R1 to renegotiate neighbor adjacencies using the new router ID.

### Verify OSPFv3

```
show ipv6 ospf interface brief
show ipv6 ospf interface <intf-id>
show ipv6 ospf neighbor
show ipv6 protocols
show ipv6 ospf
```
