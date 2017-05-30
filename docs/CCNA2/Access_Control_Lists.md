### Displaying and Verifying Access Control Lists
```
show access-lists
show access list <number>
show ip interface       #  Shows the interface/direction that ACL is applied
show running-config | include access-list 10
```

### Creating and Applying Standard IPv4 ACLs to Interfaces

1. Create the standard ACL
```
access-list <access-list-num> { deny | permit | remark } <source> [ source-wildcard ] [ log ]

access-list 10 permit host 192.168.10.10        # Specific host
access-list 10 permit 192.168.10.0 0.0.0.255    # Range of hosts
```

It is good practice to include remarks with ACLs (max 100 chars):

```
access-list 10 remark Permit hosts from x.x.x.x LAN
```


2. Activate the ACL on an interface

```
interface <interface-id>
  ip access-group { access-list-num | access-list-name } { in | out }

interface serial 0/0/0
  ip access-group 10 out
```

### Remove an access list

Removing access-list 11 from S0/0/0:

```
interface s0/0/0
no ip access-group 11 out
```

Remove the ACL:

```
no access-list 11
```

### Named Standard IPv4 ACLs

1. Create a named ACL

```
ip access-list [ standard | extended ] <name>
```

2. Specify conditions within the named ACL Configuration mode

```
[permit | deny | remark] {source [source-wildcard]) [log]
```

3. Apply the ACL to an interface

```
interface <interface-id>
ip access-group <name> [ in | out ]
```

Example:

```
ip access-list standard NO_ACCESS
  deny host 192.168.11.10
  permit any
  exit
interface g0/0
  ip access-group NO_ACCESS out
```

### ACL Statistics

- Both `permit` and `deny` statements will track statistics for matches.
- Implcit `deny any` will not appear in `show access-lists`.

To clear access list counters:

```
clear access-list counters <access-list-num>
```

### Restricting VTY Access with `access-class`

- Use with SSH to further improve administrative access security.

```
access-class <access-list-num> { in [ vrl-also ] | out }

```

Example:

```
line vty 0 4
  login local
  transport input ssh
  access-class 21 in
  exit

access-list 21 permit 192.168.0 0.0.0.255
access-list 21 deny any
```

Considerations:

- Both named and numbered access lists can be applied to VTYs.
- Identical restrictions should be set on all the VTYs, because a user can attempt to connect to any of them.
- Access lists apply to packets that travel through a router. They are not designed to block packets that originate within the router. By default, an outbound ACL does not prevent remote access connections initiated from the router.

### Troubleshooting Standard IPv4 ACLs

Common errors include:

- Entering ACEs in the wrong order.
- Not specifying adequate ACL rules.
- Wrong direction, interface or source address.
