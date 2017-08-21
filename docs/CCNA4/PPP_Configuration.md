
### HDLC Encapsulation

- Default encapsulation method used by Cisco devices on synchronous serial lines.
- For connecting non-Cisco devices, uses synchronous PPP.

```
interface s0/0/0
  encapsulation hdlc
```

### PPP Configuration

Set PPP for encapsulation:

```
interface s0/0/0
  encapsulation ppp
```

Configure optional compression:

```
  compress [predictor | stac]
```

- `predictor` specifies predictor compression algorithm to be used.
- `stac` specifies Stacker (LZS) compression algorithm will be used.

Configure optional link quality monitoring:

```
  ppp quality <percentage>
```

- Line will go down if the link quality percentage is not being maintained.
- Link quality is calculated by comparing the total number of packets and bytes sent, to the total number of packets and bytes received by the destination node.

### PPP Multilink Configuration

Create a multilink bundle:

```
interface Multilink 1
 ip address <...>
 ipv6 address <...>
 ppp multilink
 ppp multilink group <number>
```

Assign interfaces to multilink bundle:

```
interface <intf-id>
 no ip address
 encapsulation ppp
 ppp multilink
 ppp multilink group <number>
```

Disable PPP multilink:

```
no ppp multlink   ! On each bundled interfaces
```

Verify PPP multilink:

```
show ppp multilink
```

### PPP Authentication

```
ppp authentication { chap | pap | \
                     pap chap | chap pap }
```

Note that the hostname and password of target router must match the username-password of the communicating router for it to work.

##### PPP PAP Authentication

R1:

```
username R3 secret class
interface s0/0/0
  encapsulation ppp
  ppp authentication pap
  ppp pap sent-username R1 password cisco
```

R3:

```
username R1 secret cisco
interface s0/0/0
  encapsulation ppp
  ppp authentication pap
  ppp pap sent-username R3 password class
```

##### PPP CHAP Authentication

R1:

```
hostname R1
username R2 password <password>

interface <intf-id>
 ip address ...
 ipv6 address ...
 encapsulation ppp
 ppp authentication chap
```

R2:

```
hostname R2
username R1 password <password>

interface <intf-id>
 ip address ...
 ipv6 address ...
 encapsulation ppp
 ppp authentication chap
```

##### PPP CHAP Authentication through the interface:

R1:

```
hostname R1

username AuthUser2 password cisco

interface <intf-id>
 ip address ...
 encapsulation ppp
 ppp authentication chap
 ppp chap hostname AuthUser1
 ppp chap password cisco
```

R2:

```
hostname R2

username AuthUser1 password cisco

interface <intf-id>
 ip address ...
 encapsulation ppp
 ppp authentication chap
 ppp chap hostname AuthUser2
 ppp chap password cisco
```

### Troubleshooting PPP Serial Encapsulation

```
debug ppp { packet |
            negotiation |
            error |
            authentication |
            compression |
            cbcp }
```


