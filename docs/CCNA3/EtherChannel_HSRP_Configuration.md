### EtherChannel with LACP Configuration

Configure FastEthernet 0/1 and 0/2 as an EtherChannel in trunk mode:

```
interface range f0/1-2
  shutdown
  channel-group <identifier> mode active
  !channel-group <identifier> mode desirable
  no shutdown

interface port-channel <identifier>
  switchport mode trunk
  switchport trunk allowed vlan 1,2,20
```

### Verifying EtherChannel

```
show interfaces port-channel <identifier>
show etherchannel summary
show etherchannel port-channel
show interfaces etherchannel
```

### Troubleshooting EtherChannel

```
show etherchannel summary
show run | begin interface port-channel
```

# HSRP Configuration Commands

Complete the following steps to configure HSRP in interface command mode:

**Step 1.** Configure HSRP version 2.

```
standby version 2
```

**Step 2.** Configure the virtual IP address for the group.

```
standby [group-num; default=0] <ip-address>
```

**Step 3.** Configure the priority for the desired active router to be greater than 100.

```
standby [group-num] priority [priority-value; default=100]
```

- HSRP priority can be used to determine the active router. The router with the highest HSRP priority will become the active router. **By default, the HSRP priority is 100**. If the priorities are equal, the router with the numerically highest IPv4 address is elected as the active router.
- By default, **after a router becomes the active router, it will remain the active router even if another router comes online with a higher HSRP priority**.

**Step 4.** Configure the active router to preempt the standby router in cases where the active router comes online after the standby router.

- Preemption is the ability of an HSRP router to trigger the re-election process. With preemption enabled, a router that comes online with a higher HSRP priority will assume the role of the active router.

```
standby [group-num] preempt
```

**Example:**

R1 (Active Router):

```
interface g0/1
  ip address 172.16.10.2 255.255.255.0
  standby version 2
  standby 1 ip 172.16.10.1
  standby 1 priority 150
  standby 1 preempt
  no shut
```

R2 (Standby Router):

```
interface g0/1
  ip address 172.16.10.3 255.255.255.0
  standby version 2
  standby 1 ip 172.16.10.1
  no shut
```

### Verify/Debug HSRP State

```
show standby
show standby brief

debug standby errors
debug standby events
debug standby packets
debug standby terse
```

