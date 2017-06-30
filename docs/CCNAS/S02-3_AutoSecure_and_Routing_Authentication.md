### Discovery Protocols CDP and LLDP

```
show cdp neighbors detail
show lldp neighbors detail
```

### Cisco AutoSecure

```
auto secure

auto secure [no-interact|full] [forwarding|management] [ntp|login|ssh|firewall|tcp-intercept]

auto secure full            ! Default interactive mode
auto secure no-interact     ! Uses recommended Cisco default
```

### OSPF MD5 Routing Protocol Authentication

Check OSPF configuration:

```
show run | begin router ospf
```

Global authentication on all OSPF enabled interfaces:

```
ip ospf message-digest-key <key> md5 <password>
area <area-id> authentication message-digest
```

Example, configure on both routers connected via s0/0/0:

```
interface s0/0/0
 ip ospf message-digest-key 1 md5 cisco12345
 ip ospf authentication message-digest
```

- Note that OSPF neighbors should have the same key and password.


### OSPF SHA Routing Protocol Authentication

Specify a SHA authentication key chain:

```
key chain <name>
 key <key-id>
  key-string <password-string>
  cryptographic-algorithm hmac-sha-256
  exit

send-lifetime start-time {infinite|end-time|duration <seconds>}     ! Optional
```

Assign the authentication key chain to the desired interfaces:

```
interface <type-number>
 ip ospf authentication key-chain <name>
```
