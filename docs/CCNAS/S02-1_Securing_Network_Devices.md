### Increasing Access Security

```
security passwords min-length <length>
service password-encryption
enable secret <password>
line vty 0 4
 exec-timeout <mins> <seconds>      ! Default is 10 minutes
line console 0
 exec-timeout <mins> <seconds>
```

##### Disable EXEC process for a specific line

- Allows only an outgoing connection on the line, disabling the EXEC process for connections that may attempt to send unsolicited data to the router.

```
line aux 0
 no exec
```

### Secret Password Algorithms

```
enable algorithm-type {md5 | scrypt | sha256} secret <unencrypted-password>
```

- `md5`: type 5
- `scrypt`: type 9
- `sha256`: type 8 - Password-based key derivation function 2 (PBKDF2) with SHA-256

```
username <username> algorithm-type {md5 | scrypt | sha256} secret <unencrypted-password>
```

### Securing Line Access

Use only `login local` and SSH for vty:

```
username Bob algorithm-type scrypt secret cisco54321
line con 0
 login local
 exit
line aux 0
 login local
 exit
line vty 0 4
 login local
 transport input ssh
```

### Banner Configuration

```
banner {motd | exec | login} <delimiter><message...><delimiter>
```

### Login Enhancement Configuration

```
login block-for <seconds> attempts <tries> within <seconds>
login quiet-mode access-class {acl-name|acl-number}
login delay <seconds>       ! Default is 1 second
login on-success log [every <login>]
login on-failure log [every <login>]
```

- Note that `login block-for` should be issued before other login commands.
- Helps provide DoS detection and prevention.

Examples

```
login block-for 15 attempts 5 within 60
ip access-list standard PERMIT-ADMIN
 remark Permit only Administrative hosts
 permit 192.168.10.10
 permit 192.168.11.10
 exit
login quiet-mode access-class PERMIT-ADMIN
login delay 10
login on-success log
login on-failure log
```

- When quiet mode is enabled, all login attempts, including valid administrative access, are not permitted, but is overridden using `login quiet-mode access-class <ACL>` for a created ACL.

### Logging Failed Attempts

```
login on-success log [every <login>]
login on-failure log [every <login>]
security authentication failure rate threshold-rate log

show login
show login failures
```
