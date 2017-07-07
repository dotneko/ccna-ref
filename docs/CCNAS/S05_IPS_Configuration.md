### Steps in Implementng IOS IPS

1. Download the IOS IPS files.
2. Create an IOS IPS configuration directory in Flash.
3. Configure an IOS IPS crypto key.
4. Enable IOS IPS.
5. Load the IOS IPS signature package to the router.

__Step 1. Download the IOS IPS files.__

[Cisco IOS Intrusion Prevention System Feature Software: Signature Data Files](https://software.cisco.com/download/type.html?mdfid=281442967&catid=null)

- `IOS-Sxxx-CLI.pkg` - The latest signature package.
- `realm-cisco.pub.key.txt` - The public crypto key used by IOS IPS.

__Step 2. Create an IOS IPS configuration directory in Flash.__

```
mkdir IPSDIR
dir flash:
```

__Step 3. Configure an IOS IPS Crypto Key__

Located in the `realm-cisco.pub.key.txt` file that was obtained in Step 1.

The crypto key verifies the digital signature for the master signature file (`sigdef-default.xml`). The content of the file is signed by a Cisco private key to guarantee its authenticity and integrity.

Open the text file to configure the IOS IPS crypto key, as shown in Figure 1. Copy the contents of the file, and paste the contents to the router at the global configuration prompt. The *text file issues the various commands to generate the RSA key*.

At the time of signature compilation, an error message is generated if the public crypto key is invalid.

Example Error Message:

```
%IPS-3-INVALID_DIGITAL_SIGNATURE: Invalid Digital Signature found (key not found)
```

If the key is configured incorrectly, the key must be removed and then reconfigured. Use the `no crypto key pubkey-chain rsa` and the` no named-key realm-cisco.pub signature` commands. Then repeat the procedure in Step 3 to reconfigure the key.

__Step 4. Enable IOS IPS__

**a. Identify the IPS rule name and specify the location.**

Create a rule name. An optional extended or standard ACL can be configured to filter the scanned traffic. All traffic that is permitted by the ACL is subject to inspection by the IPS. Traffic that is denied by the ACL is not inspected by the IPS.

```
ip ips name <rule-name>
ip ips name <rule-name> list <extended or standard acl>
ip ips config location flash:IPS
```

Note: Prior to IOS 12.4(11)T, the `ip ips sdf` location command was used instead of `ip ips config location`.

**b. Enable SDEE and logging event notification.**

To use SDEE, the HTTP or HTTPS server must first be enabled with the `ip http server` or `ip https server` command. If the HTTP server is not enabled, the router cannot respond to the SDEE clients because it cannot see the requests.

Note:
- **SDEE notification is disabled by default and must be explicitly enabled**.
- SDEE and logging can be used independently or enabled at the same time.
- Logging notification is enabled by default.

```
ip http server
ip ips notify ?
ip ips notify sdee  ! Send msgs in SDEE format
ip ips notify log   ! Send msgs in syslog format (default)
```

**c. Configure the signature category.**

All signatures are grouped into categories, and the categories are hierarchical. This helps classify signatures for easy grouping and tuning. The three most common categories are `all`, `basic`, and `advanced`.

*When IOS IPS is first configured, all signatures in the all category should be retired*. Then, selected signatures should be unretired in a less memory-intensive category.

```
ip ips signature-category
  category all
    retired true
    exit
  category ios_ips ?
  category ios_ips basic
    retired false
    end
! Prompt: Confirm changes ?
```

CAUTION:
- **Do not unretire the all category**. The all signature category contains all signatures in a signature release. The IOS IPS cannot compile and use all the signatures at one time because it will run out of memory.
- The order in which the signature categories are configured on the router is also important.

**d. Apply the IPS rule to a desired interface, and specify the direction.**

Use the `ip ips interface configuration` command to apply the IPS rule, shown in Figure 4.

In the example, the IPS rule IOSIPS is applied to incoming traffic on the G0/0 interface. It is also applied to the incoming and outgoing traffic on the G0/1 interface.

```
interface g0/0
  ip ips IOSIPS in
  exit
interface g0/1
  ip ips IOSIPS in
  ip ips IOSIPS out
  end
```

__Step 5. Loading IOS IPS Signature Package to the Router__

Example:

```
copy tftp://192.168.1.3/IOS-S415-CLI.pkg idconf
copy ftp://ftp_user:password@<server-ip-address>/signaturefile.pkg idconf
```

To verify that the signature package is properly compiled:

```
show ip ips signature count
```

### Retire and Unretire Signatures

Example, the signature 6130 with subsig ID of 10 is retired:

```
ip ips signature-definition
  signature 6130 10
    status
      retired true
      exit
    exit
  exit
! Prompt: Confirm changes ?
```

In this example, all signatures that belong to the IOSIPS Basic category are unretired:

```
ip ips signature-category
  category ios_ips basic
    retired false
    exit
  exit
! Prompt: Confirm changes ?
```

### Enable Signature

```
ip ips signature-definition
  signature <id> <sub-id>
    status
      enabled true
    exit
  exit
exit
```

### Change Signature Actions

To change an action, the `event-action` command must be used in IPS Category Action mode or Signature Definition Engine mode.

The `event-action` command has several parameters:

```
event-action <action>
```

- `deny-attacker-inline    `: Terminates current/future packets from this attacker addr
- `deny-connection-inline  `: Terminates current/future packets on this TCP flow
- `deny-packet-inline      `: Terminates the packet
- `produce-alert           `: Writes event to the Event Store as an alert
- `reset-tcp-connection    `: Sends TCP resets to hijack and terminate TCP flow

Example - Change Actions for Specific Signature:

```
ip ips signature-definition
  signature 6130 10
    engine
      event-action produce-alert
      event-action deny-packet-inline
      event-action reset-tcp-connection
      exit
    exit
  exit

! Prompt: Confirm changes ?
```

Example - Change Actions for a Category:

```
ip ips signature-category
  category ios_ips basic
    event-action produce-alert
    event-action deny-packet-inline
    event-action reset-tcp-connection
    exit
  exit
! Prompt: Confirm changes ?
```

### Verify IOS IPS

After IPS is implemented, it is necessary to verify the configuration to ensure correct operation. There are several `show` commands that can be used to verify the IOS IPS configuration:

The `show ip ips` privileged EXEC command can be used with other parameters to provide specific IPS information.
The `show ip ips all` command displays all IPS configuration data, as shown in Figures 1 and 2. The output can be lengthy depending on the IPS configuration.
The `show ip ips configuration` command displays additional configuration data that is not displayed with the show running-config command. Figure 3 displays example output of the command.
The `show ip ips interfaces` command displays interface configuration data, as shown in Figure 4. The output shows inbound and outbound rules applied to specific interfaces.
The `show ip ips signatures` command verifies the signature configuration, as shown in Figure 5. The command can also be used with the keyword detail to provide more explicit output.
The `show ip ips statistics` command displays the number of packets audited, and the number of alarms sent, as shown in Figure 6. The optional `reset` keyword resets output to reflect the latest statistics.
Use the `clear ip ips` configuration command to disable IPS, remove all IPS configuration entries, and release dynamic resources. The `clear ip ips statistics` command resets statistics on packets analyzed, and alarms sent.

```
show ip ips ?
show ip ips all
show ip ips configuration
show ip ips interfaces
show ip ips signatures
show ip ips signatures | begin SigID
show ip ips statistics
show ip ips statistics reset
clear ip ips
clear ip ips statistics
```

### Report IPS Alerts

To specify the method of event notification, use the `ip ips notify` global configuration mode command. The `log` keyword sends messages in syslog format. The `sdee` keyword sends messages in SDEE format.

The example in the figure enables syslog reporting.

```
config t
  logging 192.168.10.100
  ip ips notify log
  logging on
```

### Enable SDEE

SDEE is the preferred method of reporting IPS activity. SDEE uses *HTTP and XML* to provide a standardized approach. It can be enabled on an IOS IPS router using the `ip ips notify sdee` command. The Cisco IOS IPS router can still send IPS alerts via syslog.

The figure shows an example of enabling SDEE reporting.

```
ip http server
ip http server-secure
ip ips notify sdee
ip sdee events <buffer-size, default:200, max:1000>
```

Clearing SDEE events/subscriptions:

```
clear ip ips sdee { events | subscriptions }
```

Note: The` ip ips notify` command replaces the older `ip audit notify` command. If the `ip audit notify` command is part of an existing configuration, the IPS interprets it as the `ip ips notify` command.
