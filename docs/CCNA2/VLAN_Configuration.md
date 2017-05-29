# Configuring VLANs

- Configurations are stored in the `flash:vlan.dat` file.

### Display/Verify VLANs

```
show vlan
show vlan brief
show vlan summary
show vlan id <vlan-id>
show vlan name <vlan-name>

show interfaces vlan <vlan-id>
show interfaces <interface-id> switchport
```

### Create a VLAN

```
vlan <vlan-id>
name <vlan-name>
exit

vlan 100,102, 105-107		# Creates multiple VLANs
```

### Assigning Ports to VLANs

```
interface <interface-id>
  switchport mode access
  switchport access vlan <vlan-id>
```

- `interface range <interfaces>` can be used to configure multipe interfaces.


### Changing VLAN Port Membership

```
interface <interface-id>
  no switchport access vlan	# Removes vlan; not necessary if want to replace
```

### Deleting VLANs

```
no vlan <vlan-id>		# Delete single VLAN

delete flash:vlan.dat		# Deletes entire vlan.dat file
```

- Note: before deleting a VLAN, reassign all member ports to a different VLAN first. Any ports that are not moved to an active VLAN are unable to communicate with other hosts after the VLAN is deleted and until tey are assigned to an active VLAN.

### Restore switch to factory defaults

```
erase startup-config
delete vlan.dat
reload
```

# Configuring VLAN Trunks

```
interface <interface-id>
  switchport mode trunk
  switchport trunk native vlan <vlan-id>
  switchport trunk allowed vlan <vlan-list>
exit
```
### Resetting the Trunk to Default State

```
interface <interface-id>
  no switchport trunk allowed vlan
  no switchport trunk native vlan
end
```

# Troubleshooting

### Troubleshooting VLANs

```
show vlan
show mac address-table
show interfaces
show interfaces switchport
show interface <interface-id> switchport
```

- Each VLAN must correspond to a unique IP subnet; check IP addressing.
- Check that ports belongs to expected VLANs and are active.
- Check if inactive VLAN is assigned to a port: `show interfaces switchport`

### Troubleshooting Trunks

```
show interfaces trunk
```

- Check whether local and peer native VLANs match (VLAN leaking occurs with mismatch).
- Check whether trunk has been established. Cisco Catalyst switch ports use DTP by default; statically configure trunk links whenever possible.
- Check status of trunk ports for incorrect port modes
- Common configuration errors:
    - Native VLAN mismatches: inter-VLAN routing issues; security risk.
    - Trunk mode mismatches: trunk link will not work.
    - Allowed VLANs on trunks: unexpected or no traffic.

# Inter-VLAN Routing

### Configure Legacy Inter-VLAN Routing

Scenario:
- R1 connected to switch ports: G0/0 -> F0/4 (VLAN 10), G0/1 -> F05 (VLAN 30)
- S1 switch ports F0/4 and F0/11 belong to VLAN 10; F0/5 and F0/6 belong to VLAN 30.

Configure the switch:

```
vlan 10					# Create VLAN 10
vlan 30					# Create VLAN 30
interface f0/11
  switchport access vlan 10
interface f0/4
  switchport access vlan 10
interface f0/6
  switchport access vlan 30
interface f0/5
  switchport access vlan 30
  end
```

Configure the router:

```
interface g0/0
  ip address <default gateway of VLAN10> <subnet-mask>
  no shutdown
interface g0/1
  ip address <default gateway of VLAN 30> <subnet-mask>
  no shutdown
  end
```

Check routing table with:

```
show ip route
```

### Configure Router-on-a-Stick

Scenario:
- R1 is connected to switch S1 on trunk port F0/5
- VLANs 10 and 30 are added to switch S1
- Two subinterfaces are configured using the `interface <interface-id>.<subinterface-id>`
	- G0/0.10: 172.17.10.1/24
	- G0/0.30: 172.17.30.1/24

Configure the switch:

```
vlan 10
vlan 30
interface f0/5
  switchport mode trunk
  end
```

Configure the router:

```
interface g0/0.10
  encapsulation dot1q 10
  ip address 172.17.10.1 255.255.255.0
interface g0/0.30
  encapsulation dot1q 30
  ip address 172.17.30.1 255.255.255.0
interface g0/0
  no shutdown
```

To set the IEEE 802.1Q native VLAN, use the `native` keyword option; by default, the native VLAN is VLAN 1.
If the physical interface is disabled, all subinterfaces are disabled.

Verify Subinterfaces:

```
show vlan
show ip route
```

Verify routing:

```
ping <ip-address>
traceroute <ip-address>		# UNIX
tracert <ip-address>		# Windows
```