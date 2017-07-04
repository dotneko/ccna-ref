### IPv6 ACL Syntax

The ACL functionality in IPv6 is similar to ACLs in IPv4. However, there is no equivalent to the IPv4 standard ACLs and **all IPv6 ACLs must be configured with a name**. IPv6 ACLs allow filtering based on *source and destination addresses* that are traveling inbound and outbound to a specific interface. They also support traffic filtering based on *IPv6 option headers and optional, upper-layer protocol type information* for finer granularity of control, similar to extended ACLs in IPv4. To configure an IPv6 ACL, use the `ipv6 access-list` command to enter into IPv6 ACL configuration mode. Next, use the syntax shown in the figure to configure each access list entry to specifically permit or deny traffic. Apply an IPv6 ACL to an interface with the `ipv6 traffic-filter` command.

```
ipv6 access-list <ACL name>
  deny | permit <protocol> \
  {<source-ipv6-prefix/prefix-length> | any | host <source-ipv6-address>} \
  [operator [port-number]] \
  {<destination-ipv6-prefix/prefix-length> | any | host <destination-ipv6-address>} \
  [operator [port-number]]
```

- Operator: compares source/destination ports
    - `lt`: less than
    - `gt`: greater than
    - `eq`: equal
    - `neq`: not equal
    - `range`
- Port-number: decimal number or name of TCP/UDP port for filtering TCP or UDP, respectively.

### Configure IPv6 ACLs

An IPv6 ACL contains an **implicit** `deny ipv6 any`. Each IPv6 ACL also contains implicit permit rules to enable **IPv6 neighbor discovery**. The IPv6 NDP requires the use of the IPv6 network layer to send neighbor advertisements (NAs) and neighbor solicitations (NSs). If an administrator configures the `deny ipv6 any` command *without explicitly permitting neighbor discovery, then the NDP will be disabled*.

Example: R1 (connects to the Internet) is permitting inbound traffic on G0/0 from the 2001:DB8:1:1::/64 network:

```
ipv6 access-list LAN_ONLY
  permit 2001:db8:1:1::/64 any
  permit icmp any any nd-na
  permit icmp any any nd-ns
  deny ipv6 any any
  end
```
- Any NA and NS packets are explicitly permitted. Traffic sourced from any other IPv6 address is explicitly denied. If the administrator only configured the first permit statement, the ACL would have the same effect. However, it is a good practice to document the implicit statements by explicitly configuring them.

Display/verify access-list:

```
show ipv6 access-list LAN_ONLY
```
