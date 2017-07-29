
### VTP Configuration

Steps:

1. Configure the VTP Server
2. Configure the VTP Domain Name and Password
3. Configure the VTP Clients
4. Configure VLANs on the VTP Server
5. Verify the VTP Clients Have Received the New VLAN Information

**Step 1: Configure VTP Server**

```
conf t
vtp mode ?
vtp mode server

show vtp status
```

**Step 2: Configure VTP Domain Name and Password**

```
vtp domain <domain-name>
vtp password <password>

show vtp password
```

**Step 3: Configure the VTP Clients**

```
vtp mode client
vtp domain <domain-name>
vtp password <password>
```

**Step 4: Configure VLANs on the VTP Server**

Example:

```
vlan 10
 name SALES
vlan 20
 name MARKETING
vlan 30
 name ACCOUNTING
```

**Step 5: Verify that the VTP Clients Have Received the New VLAN Information**

```
show vlan brief
show vtp status
```

### VLAN Configuration

```
vlan <vlan-id>
 name <name>
exit

interface <interface-id>
 switchport mode access
 switchport access vlan <vlan-id>

show vlan
show vlan brief
show vlan name <name>
show interfaces vlan <vlan-id>
```

### Deleting VLANs

On occasion, you have to remove a VLAN from the VLAN database.

- __When deleting a VLAN from a switch that is in VTP server mode__, the *VLAN is removed from the VLAN database for all switches in the VTP domain*.
- __When you delete a VLAN from a switch that is in VTP transparent mode__, the *VLAN is deleted only on that specific switch or switch stack*.

Note: You *cannot delete the default VLANs* (i.e., VLAN 1, 1002 - 1005).

The following scenario illustrates how to delete a VLAN. Assume S1 has VLANs 10, 20, and 99 configured, as shown in Figure1. Notice that VLAN 99 is assigned to ports Fa0/18 through Fa0/24.

To delete a VLAN, use the `no vlan <vlan-id>` global configuration mode command. Figure 2 demonstrates how to delete VLAN 99 from S1 and verify that VLAN 99 is gone.

**When you delete a VLAN, any ports assigned to that VLAN become inactive**. They remain associated with the VLAN (and thus inactive) until you assign them to a new VLAN.

### DTP Negotiated Interface Modes

Ethernet interfaces on Catalyst 2960 and Catalyst 3560 Series switches support different trunking modes with the help of DTP:

- `switchport mode access` - Puts the interface (access port) into permanent nontrunking mode and negotiates to convert the link into a nontrunk link. The interface becomes a nontrunk interface, regardless of whether the neighboring interface is a trunk interface.
- `switchport mode dynamic auto` - Makes the interface able to convert the link to a trunk link. The interface becomes a trunk interface if the neighboring interface is set to trunk or desirable mode. **The default switchport mode for all Ethernet interfaces is dynamic auto**.
- `switchport mode dynamic desirable` - Makes the interface actively attempt to convert the link to a trunk link. The interface becomes a trunk interface if the neighboring interface is set to trunk, desirable, or dynamic auto mode. This is the *default switchport mode on older switches*, such as the Catalyst 2950 and 3550 Series switches.
- `switchport mode trunk` - Puts the interface into permanent trunking mode and negotiates to convert the neighboring link into a trunk link. The interface becomes a trunk interface even if the neighboring interface is not a trunk interface.
- `switchport nonegotiate` - Prevents the interface from generating DTP frames. You can use this command only when the interface **switchport mode is access or trunk**. You must manually configure the neighboring interface as a trunk interface to establish a trunk link.


Verify DTP Configuration:

```
show dtp interface
```

### Extended VLAN Configuration

- Extended range VLANs are identified by a VLAN ID between 1006 and 4094.
- *By default, a Catalyst 2960 Plus Switch does not support extended VLANs*.

In order to configure an extended VLAN on a 2960 switch it **must be set to VTP transparent mode**:

```
vtp mode transparent
vlan <extended-vlan-num>
```

### Troubleshooting

```
show interfaces
show ip interface
```

Common problems with VTP are:

**Incompatible VTP Versions**

- VTP versions are incompatible with each other.
- Ensure that all switches are capable of supporting the required VTP version.

**VTP Password Issues**

- If VTP authentication is enabled, switches must all have the same password configured to participate in VTP.
- Ensure that the password is manually configured on all switches in the VTP domain.

**Incorrect VTP Domain Name**

- An improperly configured VTP domain affects VLAN synchronization between switches and if a switch receives the wrong VTP advertisement, the switch discards the message.
- To avoid incorrectly configuring a VTP domain name, **set the VTP domain name on only one VTP server switch**.
- All other switches in the same VTP domain will accept and automatically configure their VTP domain name when they receive the first VTP summary advertisement.

**All Switches set to VTP Client Mode**

- If all switches in the VTP domain are set to client mode, you cannot create, delete, and manage VLANs.
- To avoid losing all VLAN configurations in a VTP domain, configure two switches as VTP servers.

**Incorrect Configuration Revision Number**

- If a switch with the same VTP domain name but a higher configuration number is added to the domain, invalid VLANs can be propagated and / or valid VLANs can be deleted.
- The solution is to reset each switch to an earlier configuration and then reconfigure the correct VLANs.
- Before adding a switch to a VTP-enabled network, reset the revision number on the switch to `0` by assigning it to another false VTP domain and then reassigning it to the correct VTP domain name.

### Layer 3 Switch Configuration

To configure routed ports, use the `no switchport` interface configuration mode command on the appropriate ports. For example, the default configuration of the interfaces on Catalyst 3560 switches are Layer 2 interfaces, so they must be manually configured as routed ports. In addition, assign an IP address and other Layer 3 parameters as necessary. After assigning the IP address, verify that IP routing is globally enabled and that applicable routing protocols are configured.

Note: Routed ports are not supported on Catalyst 2960 Series switches.

### Layer 3 Switch Configuration Issues

The issues common to legacy inter-VLAN routing and router-on-a-stick inter-VLAN routing are also manifested in the context of Layer 3 switching. To troubleshoot Layer 3 switching issues, the following items should be checked for accuracy:

- **VLANs** - VLANs must be defined across all the switches. VLANs must be enabled on the trunk ports. Ports must be in the right VLANs.
- **SVIs** - SVIs must have the correct IP address or subnet mask. SVIs must be up. Each SVI must match with the VLAN number.
- **Routing** - Routing must be enabled. Each interface or network should be added to the routing protocol or static routes entered, where appropriate.
- **Hosts** - Hosts must have the correct IP address or subnet mask. Hosts must have a default gateway associated with an SVI or routed port.

To troubleshoot the Layer 3 switching problems, be familiar with the implementation and design layout of the topology.

