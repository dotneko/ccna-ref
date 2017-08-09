### PPPoE Configuration

Customer Router:

```
interface dialer 2
  encapsulation ppp
  ip address negotiated

  ! Authenticate inbound only:
  ppp chap hostname <Username on ISP Router>
  ppp chap password <Password>

  ip mtu 1492
  dialer pool <number>

  no shutdown

interface GigabitEthernet0/1
  no ip address
  pppoe enable
  pppoe-client dial-pool-number <number>
  no shutdown
```

### PPPoE Verification

```
show interface dialer <number>
show ip route
show pppoe session
```

### GRE Configuration

To implement a GRE tunnel, the network administrator must first learn the IP addresses of the endpoints. After that, there are five steps to configuring a GRE tunnel:

1. Create a tunnel interface using the `interface tunnel <number>` command.
2. Configure an IP address for the tunnel interface. This is normally a private IP address.
3. Specify the tunnel source IP address.
4. Specify the tunnel destination IP address.
5. (Optional) Specify GRE tunnel mode as the tunnel interface mode. GRE tunnel mode is the default tunnel interface mode for Cisco IOS software.


```
interface Tunnel 0
  tunnel mode gre ip
  ip address <ipv4-addr> <netmask>
  tunnel source <ipv4-addr>
  tunnel destination <ipv4-addr>

router ospf <process-id>
  network <ipv4-network> <wildcard> area <area>
```

### Verify GRE

```
show ip interface brief | include Tunnel
show interface Tunnel <num>
```

### BGP Configuration

```
router bgp <as-number>
  neighbor <ip-address> remote-as <as-number>
  network <network-address> [mask <network-mask>]
```

To advertise a default route:

```
  network 0.0.0.0
```

Note that above method is not the best way to advertise a default route in eBGP but the better ways are beyond the scope of CCNA4.

### Verify BGP

```
show ip route
show ip bgp
show ip bgp summary
```


