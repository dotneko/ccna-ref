# NAT Terminology

- **Inside address**: the address of the device which is being translated by NAT.
- **Outside address**: the address of the destination device (on another network).
- **Local address**: any address that appears on the inside portion of the network.
- **Global address**: any addresss that appears on the outside portion of the network.
- NAT includes 4 types of addresses:
    - Inside local address
    - Inside global address
    - Outside local address
    - Outside global address
- The NAT router is the demarcation point between the inside and outside networks and as between local and global addresses.

### Three Types of NAT

1. Static NAT: one-to-one address mapping between local and global addresses.
2. Dynamic NAT: many-to-many address mapping; translations on an as-available basis; limited by the availability of inside global addresses.
3. Port address translation (PAT): many-to-one mapping between local and global addresses. Also known as **overloading (NAT overloading)**. Ports are used as an addiitonal parameter to provide a multiplier effect.

### Static NAT Configuration

1. Create a mapping between inside local and inside global addresses.
2. Configure interfaces participating in the translation.

```
ip nat inside source static <inside-local ip> <inside-global ip>
interface <inside-interface-id>
  ip address <inside-router-ip> <router-network-mask>
  ip nat inside
  exit
interface <outside-interface-id>
  ip address <outside-router-ip> <network-mask>
  ip nat outside
```

### Dynamic NAT Operation

With dynamic NAT, a single inside address is translated to a single outside address. With this type of translation there must be enough addresses in the pool to accommodate all the inside devices needing access to the outside network at the same time. If all of the addresses in the pool have been used, a device must wait for an available address before it can access the outside network.

### Dynamic NAT Configuration

1. Define the pool of addresses that will be used for translation.
2. Configure a standard ACL to identify (permit) only those addresses to be translated.
3. Bind the ACL to the pool.
4. Identify which interfaces are inside/outside in relation to NAT.

```
ip nat pool <nat-pool-name> <starting-ip-address> <ending-ip-address> netmask <netmask>
access-list 1 permit <inside-network> <wildcard-mask>
ip nat inside source list <access-list-num> pool <nat-pool-name>
interface <inside-interface-id>
  ip nat inside
interface <outside-interface-id>
  ip nat outside
```

Two ways to configure PAT, depending on how ISP allocates public IPv4 addresses:

1. ISP allocates more than one public IPv4 address
2. ISP allocates a single public IPv4 address.

### PAT Configuration for a Pool of Public IPv4 Address

- Similar to dynamic NAT, except that there are not enough for a one-to-one mapping of inside to outside addresses.

```
ip nat pool NAT-POOL2 <start-ip> <end-ip> netmask <netmask>
access-list 1 permit <local-inside-network> <wildcard-mask>
ip nat inside source list 1 pool NAT-POOL2 overload
interface <interface-id>
  ip nat inside
interface <interface-id>
  ip nat outside
```

### PAT Configuration for a Single Address

1. Define a standard access list permitting the address that should be translated.
2. Establish dynamic source translation, specifying the ACL, exit interface and overload options.
3. Identify the inside interface.
4. Identify the outside interface.

```
access-list <access-list-num> permit <source> [source-wildcard]
ip nat inside source list <access-list-num> interface <type-num> overload
interface <inside-interface-id>
  ip nat inside
interface <outside-interface-id>
  ip nat outside
```

### Verify Static / Dynamic NAT / PAT

```
show ip nat translations
show ip nat translations verbose
show ip nat statistics
```

- Static translations, unlike dynamic translations, are always in the NAT table.
- To verify that NAT translation is working, it is best to clear statistics from past ranslations before testing:

```
clear ip nat statistics
```

Reconfigure translation entry time outs (usu 24hr):

```
ip nat translation timeout <timeout-seconds>
```

### Clearing NAT Translations

- Dynamic translations are cleared; static translations cannot be cleared form the translation table.
```
clear ip nat translation *
clear nat translation inside <global-ip> <local-ip> [ outside <local-ip> <global-ip> ]
clear
```

# Port Forwarding

Port forwarding can be enabled for applications by specifying the inside local address that requests should be forwarded to.

[Port forwarding guide](http://www.portforward.com)

### Port Forwarding Configuration with IOS

- Port forwarding is essentially a static NAT translation with a specified TCP or UDP port number.
- Similar to static NAT, the show ip nat translations command can be used to verify the port forwarding.

```
ip nat inside source static tcp <inside-local-ip> <source-port> <outside-network-interface-ip> <dest-port>
interface <inside-interface-id>
  ip nat inside
interface <outside-interface-id>
  ip nat outside
```
