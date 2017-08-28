# Securing the Local Network

### Common Layer 2 Attacks:

- CAM Table Attacks: incl. CAM table overflow (MAC address flooding)
- VLAN Attacks: incl. VLAN hopping, and VLAN double-tagging attacks, attacks between devices on a common VLAN
- DHCP Attacks: DHCP starvation and DHCP spoofing attacks
- ARP Attacks: ARP spoofing and ARP poisoning attacks
- Address Spoofing Attacks: MAC address and IP address spoofing attacks
- STP Attacks: Spanning Tree Protocol manipulation attacks

### Cisco Solutions to help Mitigate Layer 2 Attacks:

- IP Source Guard (IPSG): prevent MAC and IP address spoofing attacks.
- DAI (Dynamic ARP Inspection): prevents ARP spoofing and ARP poisoning attacks.
- DHCP Snooping: prevents DHCP starvation and DHCP spoofing attacks.
- Port Security: prevents many types of attacks including CAM table overflow attacks and DHCP starvation attacks.

### Countering CAM Table Attacks

Enable port-security on interface:

- Must first configure interface as an access port as default is dynamic auto (trunking on).

```
interface <intf-id>
  switchport mode access
  switchport port-security
```

Enabling specific port-security options:

```
  switchport port-security mac-address <mac-addr> \
    { vlan | [acess|voice] }
  switchport port-security mac-address sticky     ! Learn dynamically
  switchport port-security maximum <number-max-addresses; default=1>
  switchport port-security violation <mode>
```

Notes:

- Max number of MAC address allowed on the switch could be viewed via `show sdm prefer`
- Three violation modes:
    - `protect`: drops unknown source address packets when number of secure MAC addresses reaches the limit, with no notification.
    - `restrict`: drops unknown source address packets when number of secure MAC address reachest the limit and issues a *notification that a security violation has occurred*.
    - `shutdown`(default): interface immediately becomes error-disabled, port LED turns off, violation counter is incremented. Must be re-enabled manually using `shutdown` and `no shutdown`.

Re-enable error-disabled port:
- `shutdown` then `no shutdown`
- Automatically: `errdisable recovery cause psecure-violation`

### Port Security Aging


```
switchport port-security aging \
  {static | time <time> | type {absolute | inactivity} }
```

- `static`: Enable aging for statically configured secure addresses on this port.
- `time <time>`: Range 0-1440 mins; disable aging by setting `time 0`
- `type absolute`: All secure addresses on this port age out exactly after the time specified.
- `type inactivity`: All secure address on this port age out only if there is no data traffic from the secure source address from the specified time period.

Example configuration:

```
interface f0/1
  switchport port-security aging time 10
  switchport port-security aging type inactivity
  exit

snmp server enable traps port-security trap-rate 5
exit

show port-security interface f0/1
```

### Port Security with IP Phones

Note that an IP phone requires up to 2 MAC addresses, so switchports connected to IP phones will need an additional MAC address assigned; example:

```
interface f0/1
  switchport mode access
  switchport port-security
  switchport port-security maximum 3
  switchport port-security violation shutdown
  switchport port-securigy aging time 120
```

### SNMP MAC Address Notification

Enable notifications when SNMP traps are configured:

```
mac address-table notification
```

### Mitigating VLAN Hopping Attacks

- Disable DTP (auto trunking) negotiations on non-trunking ports by using the `switchport mode access` interface configuration command.
- Manually enable the trunk link on a trunking port using the `switchport mode trunk` interface configuration command.
- Disable DTP (auto trunking) negotiations on trunking ports using the `switchport nonegotiate` interface configuration command.
- Set the native VLAN to be something other than VLAN 1 and to be set on an unused VLAN using the `switchport trunk native vlan <vlan_number>` interface configuration mode command.
- Disable unused ports and put them in an unused VLAN.

### Mitigating DHCP Spoofing / DHCP Starvation Attacks

A **DHCP spoofing attack** occurs when a rogue DHCP server is connected to the network and *provides false IP configuration parameters to legitimate clients*.

A **DHCP starvation attack** occurs when an attacker attempts to lease all available IP addresses by creating DHCP discovery messages with bogus MAC addresses.

##### DHCP snooping

- DHCP spoofing attacks can be mitigated using **DHCP snooping on trusted ports**.
- DHCP snooping also helps mitigate against DHCP starvation attacks by *rate limiting the number of DHCP discovery messages that an untrusted port can receive*.
- DHCP snooping builds and maintains a **DHCP snooping binding database** that the switch can use to filter DHCP messages from untrusted sources. The DHCP snooping binding table includes the *client MAC address, IP address, DHCP lease time, binding type, VLAN number, and interface information* on each untrusted switchport or interface.

Steps:

1. Enable DHCP snooping using the `ip dhcp snooping` global configuration command.
2. On trusted ports, use the `ip dhcp snooping trust` interface configuration command.
3. Enable DHCP snooping by VLAN, or by a range of VLANs.

Example:

```
ip dhcp snooping
interface f0/1
  ip dhcp snooping trust
  exit
interface f0/5-24
  ip dhcp snooping limit rate 6
  exit
ip dhcp snooping vlan 5,10,50-52
```

Verify DHCP Snooping/Snooping Binding:

```
show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp snooping database
```

### Mitigating ARP Spoofing and ARP Poisoning Attacks

- To prevent ARP spoofing or poisoning, a switch must **ensure that only valid ARP requests and replies are relayed**.

**Dynamic ARP inspection (DAI)** helps prevent such attacks by *not relaying invalid or gratuitous ARP Replies out to other ports* in the same VLAN. Dynamic ARP inspection *intercepts all ARP Requests and all replies on the untrusted ports*. Each intercepted packet is *verified for valid IP-to-MAC binding*. ARP Replies coming from invalid devices are either dropped or logged by the switch for auditing so that ARP poisoning attacks are prevented. DAI can also be rate limited to limit the number of ARP packets, and the interface can be error-disabled if the rate is exceeded.

**DAI requires DHCP snooping**. DAI determines the validity of an ARP packet based on a **valid MAC-address-to-IP-address bindings database** that is built by DHCP snooping. In addition, to handle hosts that use statically configured IP addresses, DAI can **validate ARP packets against user-configured ARP ACLs**.

Example; set uplink port f0/24 as trusted port:

```
ip dhcp snooping
ip dhcp snooping vlan 10
ip arp inspection vlan 10

interface f0/24
  ip dhcp snooping trust
  ip arp inspection trust
  exit
```

Configure DAI to drop ARP packets when IP addresses are invalid:

```
ip arp inspection validate src-mac
ip arp inspection validate dst-mac
ip arp inspection validate ip
```

- Note that multiple entries overwrite the previous command, so enter them on the same command line for more than one validation method:

```
ip arp inspection validate src-mac dst-mac ip
```

Verify with:

```
do show run | include validate
```

### Mitigating MAC / IP address Spoofing with IPSG

- IP Source Guard (IPSG) operates just like DAI, but it *looks at every packet*, not just the ARP packets. Like DAI, IPSG also **requires that DHCP snooping be enabled**.
- IPSG is *deployed on untrusted Layer 2 access and trunk ports*. **IPSG dynamically maintains per-port VLAN ACLs (PVACL) based on IP-to-MAC-to-switch-port bindings**.

Configure on untrusted port:

```
interface range f0/1-2
  ip verify source
  exit
```

To verify, use `show ip verify source`

### Mitigating Spanning-Tree Protocol (STP) Attacks

To mitigate STP manipulation attacks, use the **Cisco STP stability mechanisms** to enhance the overall performance of the switches and to reduce the time that is lost during topology changes.

These are recommended practices for using STP stability mechanisms:

- **PortFast** - PortFast immediately brings an interface configured as an access or trunk port to the forwarding state from a blocking state, *bypassing the listening and learning states*. **Apply to all end-user ports**. PortFast should only be configured when there is a *host attached to the port*, and not another switch.
- **BPDU Guard** - BPDU guard immediately *error disables a port that receives a BPDU*. Typically *used on PortFast enabled ports. Apply to all end-user ports*.
- **Root Guard** - Root guard *prevents an inappropriate switch from becoming the root* bridge. Root guard limits the switch ports out of which the root bridge may be negotiated. *Apply to all ports which should not become root ports*.
- **Loop Guard** - Loop guard *prevents alternate or root ports from becoming designated ports because of a failure that leads to a unidirectional link*. *Apply to all ports that are or can become non-designated*.

##### Configuring PortFast:

```
spanning-tree portfast default    ! Enable on all non-trunking ports

interface <intf-id>
  spanning-tree portfast          ! Enable on an interface
```

Verify with `show running-config interface <intf-id>`

##### Configuring BPDU Guard (should always do so on PortFast-enabled ports):

```
spanning-tree portfast bpduguard default

interface <intf-id>
  spanning-tree bpduguard enable
```

Verify with:

```
show spanning-tree summary
show spanning-tree summary totals
```

##### Configuring Root Guard

```
interface <intf-id>
  spanning-tree guard root
```

To view Root Guard ports that have received superior BPDUs and are in a root-inconsistent state, use the `show spanning-tree inconsistent ports` command.

##### Configuring Loop Guard

```
spanning-tree loopguard default       ! Enable globally

interface <intf-id>
  spanning-tree loopguard default    ! Enable on an interface
```
