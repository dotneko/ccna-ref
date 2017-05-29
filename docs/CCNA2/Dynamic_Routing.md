# Configuring Dynamic Routing

### Access RIP configuration mode

```
router rip
```
- Prompt will show up as `Router(config-router)#`
- When enabling RIP, the default version is RIPv1.
- To disable and eliminate RIP, use the `no router rip` global configuration command. This command stops the RIP process and erases all existing RIP configurations.
- Upon enabling, need to advertise networks with `network <network-address>`

### Enable and Verify RIPv2

By *default, when a RIP process is configured on a Cisco router, it is running RIPv1*, as shown in Figure 1. However, even though the router only sends RIPv1 messages, it can interpret both RIPv1 and RIPv2 messages. A *RIPv1 router ignores the RIPv2 fields in the route entry*.

Use the `version 2` router configuration mode command to enable RIPv2, as shown in Figure 2. Notice how the `show ip protocols` command verifies that R2 is now configured to send and receive version 2 messages only. The RIP process now includes the subnet mask in all updates, making *RIPv2 a classless routing protocol*.

Note: Configuring `version 1` enables RIPv1 only, while configuring `no version` returns the router to the default setting of sending version 1 updates but listening for version 1 and version 2 updates.

### Propogate a Default Route

 The `default-information originate` router configuration command. This instructs R1 to originate default information, by propagating the static default route in RIP updates.

### RIP Routing Configuration Mode Commands

```
network <network-address>       # Enables RIP on all interfaces on that network.
version 2                       # Enables RIPv2
version 1                       # Enables RIPv1 only
no version                      # Returns to default setting: send v1, listen v1+v2
no auto-summary                 # Disables default automatic summarization (RIPv2)

passive-interface <interface>   # Prevent transmission of routing updates out interface
                                # However, still allows network to be advertised
passive-interface g0/0
passive-interface default       # Makes all interfaces passive
no passive-interface <intf>     # Re-enables passive transmission of routing updates
```

### Verify RIP Routing

```
show ip protocols
show ip route
```

### Propogate a Default Route

To propagate a default route in RIP, the edge router must be configured with a default static route using the `ip route 0.0.0.0 0.0.0.0` command:

```
ip route 0.0.0.0 0.0.0.0 <exit-interface> <next-hop IP>
```

Next configure router to propogate the static default route in RIP updates:

```
router rip
default-information originate
```
