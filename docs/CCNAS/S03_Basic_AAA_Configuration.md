### Local Authentication Methods

Example:

```
username JR-ADMIN algorithm-type scrypt secret <strong_password>
username ADMIN algorithm-type scrypt secret <strong_password>
aaa new-model
aaa authentication login default local-case
```
### Authentication Methods

```
aaa authentication login {default|list-name} method1...[method4]
```

Method type keywords:

- `enable`: uses enable password
- `local`: uses local username database
- `local-case`: uses case-sensitive local username authentication.
- `none`: uses no authentication
- `group radius`: uses list of all RADIUS servers
- `group tacacs+`: uses list of all TACACS+ servers
- `group <group-name>`: uses subset of RADIUS/TACACS+ servers defined by:
    - `aaa group server radius` or `aaa group server tacacs+`

### Default and Named Authentication Methods

```
! (Assume local database created)

aaa new-model
aaa authentication login default local-case enable
aaa authentication login SSH-LOGIN local-case
aaa local authentication attempts max-fail 3

line vty 0 4
 login authentication SSH-LOGIN
```

### Fine-Tuning the Authentication Configuration

```
aaa local authentications attempts max-fail <num-unsuccessful-attempts>

show aaa local user lockout
show aaa sessions
```

Unlock locked user:

```
clear aaa local user lockout <username>
```

### Showing and Debugging AAA Authentication

```
show aaa sessions
debug aaa authentication
```

Disabling debugging:

```
no debug aaa authentication
undebug all
```

### TACACS+ Server Configuration

```
aaa new-model
tacacs server <name>
 address ipv4 <ip-address>
 single-connection
 key <pass-key>
 exit
```

### RADIUS Server Configuration

```
aaa new-model
radius server <name>
 address ipv4 <ip-address> auth-port <port-num> acct-port <port-num>
 key <pass-key>
 exit
```

Note: By default, Cisco routers use port 1645 for the authentication and port 1646 for the accounting. However, IANA has reserved ports 1812 for the RADIUS authentication port and 1813 for the RADIUS accounting port. It is important to make sure these ports match between the Cisco router and the RADIUS server.

### Authentication to use the AAA Server

```
aaa authentication login default ?
aaa authentication login default <method1> [<method2> <method3> <method4>]
```

- groups can be specified in priority, e.g. `group tacacs+`, `group radius`, `local-case` etc.

Example:

```
aaa new-model

tacacs server Server-T
 address ipv4 <ip-address>
 single-connection
 key <TACACS-Password>
 exit

radius server Server-R
 address ipv4 <ip-address> auth-port 1812 acct-port 1813
 key <RADIUS-Password>
 exit

aaa authentication login default group tacacs+ group radius local-case
```

### Troubleshooting Server-Based AAA Authentication

```
debug aaa authentication
debug tacacs
debug tacacs events
debug radius
```

### AAA Authorization Configuration

```
aaa authorization {network|exec|commands <level>} {default|<list-name>} <method1...[method4]>

aaa authorization exec default ?
aaa authorization exec default group ?
```

Example:

```
username jr-admin algorithm-type scrypt secret <password>
username admin algorithm-type scrypt secret <password>
aaa new-model
aaa authorization exec default group tacacs+
aaa authorization network default group tacacs+
```

### AAA Accounting Configuration

```
aaa accounting {network|exec|connection} {default|<list-name>} {start-stop|stop-only|none} [broadcast] <method1...[method4]
```

Example:

```
! Assume usernames already created

aaa new-model
aaa authentication login default group tacacs+
aaa authorization exec default group tacacs+
aaa authorization network default group tacacs+
aaa accounting exec default start-stop default group tacacs+
aaa accounting network default start-stop group tacacs+
```

### Configuring 802.1X

```
aaa new-model
radius server CCNAS
 address ipv4 10.1.1.1 auth-port 1812 acct-port 1813
 key RADIUS-Password
 exit

aaa authentication dot1x default group radius
dot1x system-auth-control
interface f0/1
 description Access Port
 switchport mode access
 authentication port-control auto
 dot1x pae authenticator
```


