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

### SSH Configuration

```
configure terminal
ip domain-name <domain_name.com>
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
username <username> algorithm-type scrypt secret <password>
line vty 0 4
 login local
 transport input ssh
end
```

Verify SSH and display generated keys:

```
show crypto key mypubkey rsa
```

Overwriting existing key pairs:

```
crypto key zeroize rsa
```

Verify optional SSH settings:

```
show ip ssh
```

Modify default timeout interval and num retries

```
ip ssh time-out <seconds; default:120>
ip ssh authentication-retries <integer>
```

Verify status of client SSH connections:

```
show ssh
```

Connecting to an SSH server:

```
ssh -l <login-name> <ip-address>
```

# 2.2 Assigning Administrative Roles

### Privilege Level Configuration and Assignment

```
show privilege      ! Displays current privilege

privilege <mode> {level <level> | reset} [command]
```

To configure a privilege level with specific commands:

```
privilege exec level <level> [command]
```

Level 5 and SUPPORT user configuration:

```
privilege exec level 5 ping
enable algorithm-type scrypt secret level5 cisco5
username SUPPORT privilege 5 algorithm-type scrypt secret cisco5
```

Level 10 and JR-ADMIN user configuration:

```
privilege exec level 10 reload
enable algorithm-type scrypt secret level 10 cisco10
username JR-ADMIN privilege 10 algorithm-type scrypt secret cisco10
```

Level 15 and ADMIN user configuration:

```
enable algorithm-type scrypt secret level 15 cisco123
username ADMIN privilege 15 algorithm-type scrypt secret cisco123
```

Note: *Assigning a command with multiple keywords allows access to all commands that use those keywords*

### Role-Based View Configuration

```
aaa new-model
parser view SHOWVIEW
 secret cisco
 commands exec include show
 exit
parser view VERIFYVIEW
 secret cisco5
 commands exec include ping
 exit
parser view REBOOTVIEW
 secret cisco10
 commands exec include reload
 exit
```

### Role-Based Superview Configuration

```
parser view USER superview
 secret cisco
 view SHOWVIEW
 exit
parser view SUPPORT superview
 secret cisco1
 view SHOWVIEW
 view VERIFYVIEW
 exit
parser view JR-ADMIN superview
 secret cisco2
 view SHOWVIEW
 view VERIFYVIEW
 view REBOOTVIEW
```

Verify view:

```
show parser view all        ! Displays summary of all views
enable view <view-name>     ! Verify specific view

enable view USER
! then enter password

?       ! Display exec commands
show ?  ! Display show options
```

