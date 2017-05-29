# Configuring SSH

```
show ip ssh                     # Verify that switch supports SSH, shows version

ip domain-name <domain-name>    # Device must be minimally configured with a hostname
ip ssh version 2                # Recommended to only enable version 2 (more secure)
crypto key generate rsa
username <username> privilege <level> secret <password>
```

### Enable SSH protocol on VTY lines

```
line vty 0 15
transport input ssh
login local                      # Require local authentication from local username database
```

### To delete a RSA key pair

```
crypto key zeroize rsa          # SSH automatically disabled when key pair is deleted.
```

# Port Security

### Disable Unused Ports

- Good practice to disable unused ports; can configure range:

```
interface range <type module>/<first number>-<last number>
  shutdown
```

### Port Security: Operation

- All switch ports (interfaces) should be secured before the switch is deployed for production use.
    - Specify a single MAC address or a group of valid MAC addresses allowed on a port.
    - Specify that a port automatically shuts down if unauthorized MAC addresses are detected.

Static Secure MAC Address:

```
interface <interface-id>
  switchport port-security mac-address <mac-address>
```

Sticky secure MAC addresses - MAC addresses that can be dynamically learned or manually configured, then stored in the address table and added to the running configuration by stick learning:

```
switchport port-security mac-address sticky             # (Intf-config)
```

- *All sticky secure MAC addresses are added to the address table and to the running configuration*.
- Sticky secure MAC addresses can also be manually defined (also added to running-config):

```
switchport port-security mac-address sticky <mac-address>
```

If the sticky secure MAC addresses are saved to the startup configuration file, then when the switch restarts or the interface shuts down, the interface does not need to relearn the addresses. If the sticky secure addresses are not saved, they will be lost.

If sticky learning is disabled by using the `no switchport port-security mac-address sticky` interface configuration mode command, the sticky secure MAC addresses remain part of the address table, but are removed from the running configuration.

Characteristics of sticky secure MAC addresses:

- Learned dynamically, converted to sticky secure MAC addresses stored in the running-config.
- Removed from the running-config if port security is disabled.
- Lost when the switch reboots (power cycled).
- Saving sticky secure MAC addresses in the startup-config makes them permanent and the switch retains them after a reboot.
- Disabling sticky learning converts sticky MAC addresses to dynamic secure addresses and removes them from the running-config.

Note: The port security feature will not work until port security is enabled on the interface using the `switchport port-security` command.

### Port Security: Violation Modes

To change the violation mode on a switch port:

```
switchport port-security violation {protect | restrict | shutdown}
```

- **Protect** - When the number of secure MAC addresses reaches the limit allowed on the port, packets with unknown source addresses are dropped until a sufficient number of secure MAC addresses are removed, or the number of maximum allowable addresses is increased. There is no notification that a security violation has occurred.
- **Restrict** - When the number of secure MAC addresses reaches the limit allowed on the port, packets with unknown source addresses are dropped until a sufficient number of secure MAC addresses are removed, or the number of maximum allowable addresses is increased. In this mode, there is a *notification that a security violation has occurred*.
- **Shutdown** - In this (*default*) mode, a port security violation causes the *interface to immediately become error-disabled and turns off the port LED. It increments the violation counter*. When a secure port is in the error-disabled state, it can be brought out of this state by entering the `shutdown` interface configuration mode command followed by the `no shutdown` command.

### Port Security: Configuring

#### Default Port Security on a Cisco Catalyst Switch
- Port security and Sticky address learning: Disabled
- Max number of secure MAC addresses: 1
- Violation mode: Shutdown (Port shuts down when max number of secure MAC addresses exceeded).

#### Configure Port Security on an Interface

```
interface <interface_id>
switchport mode access                              # Sets interface mode to access
switchport port-security                            # Enables port security on the interface
switchport port-security maximum 10                 # Max number of secure addresses allowed
switchport port-security mac-address sticky         # Enables sticky learning
```

- Violation mode is set to shutdown by default.

### Port Security: Verifying

Verify Port Security Settings:

```
show port-security interface <interface_id>
```

Verify Secure MAC Addresses:

```
show port-security address
```

### Ports in Error Disabled State

- When a port is error disabled, it is effectively shut down and no traffic is sent or received on that port.
- Port protocol and link status is changed to down; port LED will turn off; port goes into the error disabled stated.
- `show interfaces` identifies port status as `err-disabled`
- `show port-security interface` shows port status as `secure-shutdown`

The administrator should determine what caused the security violation before re-enabling the port. If an unauthorized device is connected to a secure port, the port should not be re-enabled until the security threat is eliminated. To re-enable the port, use the `shutdown` interface configuration mode command (Figure 3). Then, use the`no shutdown` interface configuration command to make the port operational.


