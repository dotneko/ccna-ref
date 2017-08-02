# EIGRP Configuration

To configure an EIGRP autonomous system and assign a router ID:
```
router eigrp <autonomous-system-number between 1-65535>
  eigrp router-id <ipv4-address>
```

Example:

```
# R1:
router eigrp 1
  eigrp router-id 1.1.1.1
  
# R2:
router eigrp 1
  eigrp router-id 2.2.2.2
```

If a router ID is not explicitly configured, then the router would use its highest IPv4 address configured on a **loopback interface**. The advantage of using a loopback interface is that unlike physical interfaces, loopbacks cannot fail. There are no actual cables or adjacent devices on which the loopback interface depends for being in the up state. Therefore, using a **loopback address for the router ID can provide a more consistent router ID than using an interface address**.

If the `eigrp router-id` command is not used and loopback interfaces are configured, EIGRP chooses the highest IPv4 address of any of its loopback interfaces. To enable and configure a loopback interface:

```
interface loopback <number>
  ip address <ipv4-address> 255.255.255.255
```

To remove an EIGRP autonomous system:

```
no router eigrp <autonomous-system-number>
```  

### Configure EIGRP Notification Messages

By default, this is enabled:

```
eigrp log-neighbor-changes
```

### Enable EIGRP routing on an interface

Enables all interfaces on the router that belong to the classful network address:

```
router eigrp <as-number>
  network <ipv4-address>
```

Configure advertisement of specific subnets only:

```
router eigrp <as-number>
  network <ipv4-address> [<wildcard-mask>]
```

Cisco IOS versions also accepts subnet mask to be used instead, but will store the wildcard mask in the `running-config`.

### Prevent Neighbor Adjacencies

```
router eigrp <AS-number>
  passive-interface <interface-type> <interface-number>
```

Configure all interfaces as passive:

```
  passive-interface default
```

### Automatic summarization

Disabled by default since IOS 15; For router with IOS verions prior to 15, disable using:

```
router eigrp <AS-number>
  no auto-summary
```

### Verify the EIGRP Process

```
show ip protocols
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp
```

### EIGRP Composite Metric Configuration

The default k values can be changed with the `metric weights` router configuration mode command:

```
router eigrp <as-number>
  metric weights <tos> <k1> <k2> <k3> <k4> <k5>
```

Examine interface metric values:

```
show interfaces <interface-type> <interface-number>
```

- **BW** - Bandwidth of the interface (in kilobits per second).
- **DLY** - Delay of the interface (in microseconds).
- **Reliability** - Reliability of the interface as a fraction of 255 (255/255 is 100% reliability), calculated as an exponential average over five minutes. By default, EIGRP does not include its value in computing its metric.
- **Txload**, **Rxload** - Transmit and receive load on the interface as a fraction of 255 (255/255 is completely saturated), calculated as an exponential average over five minutes. By default, EIGRP does not include its value in computing its metric.

##### Bandwidth Configuration

Always verify bandwidth with the `show interfaces` command. The default value of the bandwidth may or may not reflect the actual physical bandwidth of the interface. If actual bandwidth of the link differs from the default bandwidth value, the bandwidth value should be modified.

To modify the bandwidth metric (note that it does not change the actual bandwidth of the link):

```
bandwidth <kilobits-bandwidth-value>
```

Use the `no bandwidth` command to restore the default value.

##### Delay (DLY) Metric

By default, Cisco recommends not modifying the delay parameter, unless the network administrator has a specific reason to do so.

### Calculating the EIGRP Metric

1. Determine the link with the slowest bandwidth. Use that value to calculate **bandwidth** (Reference value of 10,000,000/slowest bandwidth).
2. Determine the **delay value** for each outgoing interface on the way to the destination. Add the delay values and divide by 10 (sum of delay/10).
3. This composite metric produces a 24-bit value; however, EIGRP uses a 32-bit value. Multiplying the 24-bit value with 256 extends the composite metric into 32 bits. Therefore, **add the computed values for bandwidth and delay, and multiply the sum by 256 to obtain the EIGRP metric**.

### View EIGRP Topology Table

```
show ip eigrp topology
```

Lists all successors and FSs that DUAL has calculated to destination networks. Only the successor is installed into the IP routing table.

To view all links regardless of whether they satisfy the FC or not:

```
show ip eigrp topology all-links
```

### Debug the EIGRP Finite State Machine

Use to examine what DUAL does when a route is removed from the routing table:

```
debug eigrp fsm
```

### EIGRP for IPv6

Configure Interfaces:

```
interface s0/0/0
  ipv6 address fe80::1 link-local
  exit
interface s0/0/1
  ipv6 address fe80::1 link-local
  exit
interface g0/0
  ipv6 address fe80::1 link-local
```

Enable IPv6 Unicast Routing and configure EIGRP AS:

```
ipv6 unicast-routing
ipv6 router eigrp 1
  eigrp router-id 1.0.0.0 ! Takes precedence over loopback/physical IPv4 addresses; required if no active IPv4 address
  no shutdown
```

Assign interface to EIGRP AS
```
interface <interface-id>
  ipv6 eigrp <AS-number>
  exit
```

Passive Interface:

```
ipv6 router eigrp <AS-number>
  passive-interface <interface-id>
  end
  
show ipv6 protocols
```

Verify:

```
show ipv6 interface brief   ! Check status of both ends of link
show ipv6 eigrp neighbors   ! Verify neighbor table and establishment of adjacencies.
show ipv6 protocols         ! Verify EIGRP enabled and correct AS number
show ipv6 route             ! EIGRP for IPv6 routes are denoted with "D"
```

### EIGRP Automatic Summarization

```
router eigrp <as-number>
  auto-summary
  
  no auto-summary
```

**N.B. When manually summarizing routes, configure as follows (see 7.3.1.2 Packet Tracer):**

Example:

When configuring EIGRP on a router that has 4 subnets 10.10.0.1/24, 10.10.1.0/24, 10.10.2.0/23, 10.10.4.1/22 apart from setting `no auto-summary` and specifying the classful address under router configuration, also need to use `ip summary address <ip-addr> <netmask>` in the interface configuration to specify the range of IP addresses to be summarized.

```
router eigrp 1
 no auto-summary
 network 10.0.0.0
 
interface s0/0/0
 ip summary-address 10.10.0.0 255.255.248.0
```

### Verify EIGRP Automatic Summarization Status

```
show ip protocols
show ip route eigrp
show ip eigrp topology all-links    ! View all incoming routes
```

- The `all-links` option shows all received updates, including routes from feasible successor (FS). 

### Propagate a Default Static Route

```
ip route 0.0.0.0 0.0.0.0 serial 0/1/0
router eigrp 1
  redistribute static
  exit

! Verify

show ip route | include 0.0.0.0
show ip protocols
```

__For IPv6:__ There is no specific IPv6 command to redistribute the IPv6 default static route. The IPv6 default static route is redistributed into the EIGRP for the IPv6 domain using the same `redistribute static` command used in EIGRP for IPv4.

### EIGRP Bandwidth Utilization Configuration

```
  ip bandwidth-percent eigrp <as-number> <percent>
  ipv6 bandwidth-percent eigrp <as-number> <percent>
```

Example:

```
! R1 and R2
interface serial 0/0/0
  ip bandwidth-percent eigrp 1 40
```

### Hello and Hold Timers

Configure in interface configuration mode (config-if):

**IPv4:**

```
ip hello-interval eigrp <as-number> <seconds>
ip hold-time eigrp <as-number> <seconds>
```

**IPv6:**

```
ipv6 hello-interval eigrp <as-number> <seconds>
ipv6 hold-time eigrp <as-number> <seconds>
```

- If the Hello interval is changed, **ensure that the hold time value is equal to, or greater than, the Hello interval**. Otherwise, neighbor adjacency goes down after the hold time expires and before the next Hello interval.
- The seconds value for both Hello and hold time intervals can range from 1 to 65,535.

### Load Balancing Configuration

In EIGRP router configuration mode:

```
maximum-paths <value>
```

- The `<value>` argument refers to the number of paths that should be maintained for load balancing. If the value is set to 1, load balancing is disabled.

### Unequal Cost Load Balancing

In EIGRP router configuration mode:

```
variance <value>
```

- __If the variance is set to 1__, only routes with the same metric as the successor are installed in the local routing table.
- __If the variance is set to 2__, any EIGRP-learned route with a metric less than 2 times the successor metric will be installed in the local routing table.

To control how trafﬁc is distributed among routes when there are multiple routes for the same destination network that have different costs, use the `trafﬁc-share balanced` command. Trafﬁc is then distributed proportionately to the ratio of the costs.

### Basic EIGRP Troubleshooting Commands

```
show ip eigrp neighbors
show ip route
show ip protocols

show ipv6 eigrp neighbors
show ipv6 route
show ipv6 protocols
```

If unsuccessful ping between routers, verify Layer 1 and 2 connections to the neighbor:

```
show cdp neighbor
```