### VLAN Concepts and Configuration

- **VLAN**: a layer 3 segmentation performed at layer 2 (by a switch).
- A VLAN = a broadcast domain = an IP subnet
- Recommended to isolate IP phones in VLANs dedicated to voice devices.
- Trunk ports (tagged ports) facilitate transmission of VLAN-tagged frames between switches.
- Protocol in common usage for VLANs is IEEE 802.1Q.

### Understanding Voice VLANs

- Recommended practice to separate voice and data traffic using VLANs:
    - Provides additional security layer.
    - Simpler method to deploy Qos - prioritizing voice traffic over data.
- Secure Real-Time Protocol (SRTP) can be used to encrypt audo.
- Cisco IP Phones:
    - Hardware similar to a full Cisco switch.
    - Incoming switchport able to receive and send 802.1Q tagged packets - provides capability to establish a special **multi-VLAN access port**.
    - Connection between switch and IP phone is like a *"mini-trunk"*: the IP phone tags its own packets with the correct voice VLAN, while in-transit data packets from the PC to switch are untagged. Switch would assign the untagged packets to whatever VLAN you have configured on the switchport for data traffic, at ingress.

### Understanding the Cisco IP Phone Boot Process

1. Cisco IP phone connects to ethernet switch port; receives power via PoE if available.
2. After powering on, Cisco switch delivers voice VLAN information to IP phone using *CDP*.
3. Cisco IP phone broadcasts a *DHCP* request (within voice VLAN).
4. DHCP server responds with an IP address offer. `Option 150` is required for Cisco IP phones, which directs it to a TFTP server.
5. Cisco IP phone contacts the *TFTP server and downloads a .XML configuration file*, which  includes a list of valid call processing agents. (e.g. CUCM or CME). The configuration file contains a list of up to three CUCM server or CME IP addresses the Cisco IP phone uses for registration.
6. Cisco IP phone *attempts to register* with a call processing server from those listed in the configuration file (e.g. CUCM or CME).

### VLAN Configuration

Add VLANs to the switch:

```
vlan 10
  name VOICE
vlan 50
  name DATA
end
```

Assign ports connecting to Cisco IP phones:

```
interface range f0/2-24
  switchport mode access
  spanning-tree portfast
  switchport access vlan 50
  switchport voice vlan 10
end
```

- Note that the assumption above is that interface f0/1 is a trunk port to the router.

### Configuring a Router-based DHCP Server

```
ip dhcp excluded-address 172.16.1.1 172.16.1.9
ip dhcp excluded-address 172.16.2.1 172.16.2.9

ip dhcp pool DATA_SCOPE
  network 172.16.2.0 255.255.255.0
  default-router 172.16.2.1
  option 150 ip 172.16.1.1
  dns-server 4.2.2.2
exit
ip dhcp pool VOICE_SCOPE
  network 172.16.1.0 255.255.255.0
  default-router 172.16.1.1
  option 150 ip 172.16.1.1      ! IP address of TFTP server for config-files
  dns-server 4.2.2.2
exit
```

- Use `ip helper-address <central DHCP server IP address>` on the interface for case of when want the router to forward DHCP requests to a central DHCP server for the voice VLAN devices.
- XML configuration files are named `SEP<IP Phone MAC Address>.cnf.xml`; if none exist, a *base configuration file* named `XMLDefault.cnf.xml` is requested for *Auto-Registration*.
- To store cnf-file on an external TFTP server in CME 4.0+:

```
cnf-file location tftp://<ip address of TFTP server>
```

### Setting the Time

```
clock set hh:mm:ss MMM DD YYYY
```

### Set the Clock via NTP

```
ntp server <ip address of NTP server>
clock timezone <name> <hours from UTC>

show ntp associations
show clock
```

To allow other devices such as a CUCM server to pull date and time information from a Cisco router using NTP:

```
ntp master <stratum number>
```

- Stratum 1 time server is one that has a radio or atomic clock directly attached.
- Stratum 2 time server is one that receives its time from the stratum 1 time server.

### IP Phone Registration

- If IP phone finds an active server in the list from the configuration file, it goes through the registration process using either of (depending on firmware of IP phone):
    - Skinny Client Control Protocol (SCCP)
    - Session Initiation Protocol (SIP)

Registration process:

1. Cisco IP phone contacts call processing server and identifies itself via MAC address.
2. Call processing server looks at database and sends *operating configuration* to the phone: contains items such as directory/line numbers, ring tones, softkey layout.
3. Signaling protocol is then used for majority of phone functionality following registration (SCCP or SIP).

### Preparing the CME Router for Cisco Configuration Professional

Check for telephony service and registered ephones

```
show telephony-service
show ephone registered

!! Enable or enter telephony configuration

configure terminal
 telephony-service
```

Four key elements for getting CCP to work:

- Reachable IP address
- Level username and password
- Integrated HTTP services
- Local authentication for Telnet/SSH

```
configure terminal
interface g0/0
  ip address <ip-address> <netmask>
  no shutdown
  exit
username <name> privilege 15 secret <password>

ip http server
ip http secure-server

line vty 0 15
  login local
  transport input telnet ssh
  end
```

- [Latest version of Cisco Configuration Professional 3.x](https://software.cisco.com/download/release.html?mdfid=281795035&softwareid=282159854&release=3.3.1&relind=AVAILABLE&rellifecycle=&reltype=latest)

### Enable the CME GUI

```
ip http server
ip http path <location of CME GUI, e.g. flash:>
ip http authentication local
telephony-service
  web admin system name <username> password <password>  # Configure CME System Administrator
  dn-webedit            # Enables add directory numbers through web interface
  time-webedit          # Enables ability to set phone time for CME system
```

### Concepts of Ephones and Ephone-DNs

- `ephone`: "Ethernet phone"; refers to physical phone hardware, identified by device type and MAC address.
- `ephone-dn`: an extension and association with the user that is required to:
    - provide caller ID for internal calls
    - build the system directory
    - Allow presence status monitoring of the ephone-dn
    - Apply the ephone-dn configuration to the ephone.
    - Configuration as either single-line or dual-line:
        - Single-line: make or receive only one call at a time.
        - Dual-line: can handle two simultaneous calls; required for call waiting, conference calls, consultative transfers.

### SCCP versus SIP

- SCP phone is defined by creating an `ephone`, associated with an `ephone-dn`, configured with `telephony service`.
- SIP phone is defined using a `voice register pool`, associated with a `voice register dn`, configured with `voice register global`.

### Managing the Telephony Service via the Command Line

```
show telephony-service tftp-bindings
telephony-service ?

telephony-service	! Enters config-telephony mode
```

### Basic configuration of CME router to support Cisco IP Phones

```
telephony-service
 max-dn <max dn-numbers>
 max-ephones <max number of phones>
 ip source-address <ipv4 address of Voice VLAN default-router; used by CUCME> port 2000
 create cnf-files
```

- For `ip source-address` the default TCP port of 200 is used; can be specified with the `port 2000` option (required in Packet Tracer)
- Best option for the source address is a loopback address, as loopback interfaces are always up.

### Configure auto-registration and auto-dn-assignment:

```
telephony-service
 auto assign <start-dn> to <end-dn>
 auto-reg-ephone
```

### Configuring E-Phone Directory Numbers (ephone-dn)

- `Ephone-dn`s can be single-line, dual-line or in CUCME 4.1, octo-line.
- Most desk phones use dual-lines to allow call waiting and transferring a caller to a third party, as need 2 audio channels.

```
ephone-dn 1 dual-line
  number <extension number>
  name <firstname> <lastname>		# Optional
  exit
```

```
ephone-dn <ephone-dn-tag-num>     # Enters configuration mode for ephone-dn

ephone-dn 1
 number 1001
 exit
ephone-dn 2 dual-line
 number 1002 secondary 2041112222
 exit
ephone-dn 3
 number 1003
 label <name>
 description <e.g Primary Line>
 exit
```

Configure ePhone Text Display (config-ephone-dn)

```
ephone-dn <tag-num>
 description <Phone Description>			# Information in header bar
 label <username> (extension-number)		# Assignes label to text for the line button
```

### Configuring Ephones

```
ephone <ephone-tag-num>        # Enters configuration mode for ephone
 mac-address <mac-address-of-phone>
 button <line-button>:<ephone-dn num>	# Ties first line button to first ephone-dn

! Additional options

 button <line-button><option><ephone-dn num>		# Can add options
 username <username> password <password>            # Assigns user/pass
```

- Options for `<option>` above:
   - `f`: feature ring - stutter triple ring
   - `s`: silent ring - allows to answer or use a line without having it ring
   - `m`: monitor line - allows to monitor status of an extension; button can be used to transfer a call to the extension
   - e.g. `button 2f4`: assigns button 2 to have feature ring for ephone-dn 4's extension.

Example Ephone Configuration:

```
ephone 1
 mac-address <xxxx.xxxx.xxxx>
 type 7940
 auto-line
 button 1:1         # Associates physical button 1 on phone to ephone-dn 1
 exit

ephone 1
 button 1:2 2:1     # Line 1 button assigned to ephone-dn2; Line 2 to ephone-dn1
 restart            # Updates the phone configuration
 reset              # Tells phone to completely reboot
```

Assign Users to the Phones

```
ephone <num-tag>
 username <username> password <password>
 pin 1234
```

### Miscellaneous Commands

Resetting default template files manually - if changes to config do not show on the phones:

```
no create cnf-files
create cnf-files
```

To inspect IP Phone Generic Config File

```
show telephony-service tftp-bindings
```

- Look for `XMLDefault.cnf.xml` file
- Phone usually requests an `SEP<phone-mac-address>.cnf.xml` file that contains settings the phone should use.

Configure System Time in Telephony-Service

```
telephony-service
 time-zone ?			# Displays list of timezones
 time-zone <timezone num>
```

Configure Banner Message for Phones with a Display

```
telephony-service
 system message ?
 system message Pod 1	# Displays pod number (config-telephony)
```

### Verification and Troubleshooting

```
show ephone attempted-registrations
show ephone registered
show ephone
```

### Understanding the CME Dial Plan

##### Configuring Physical Voice Port Characteristics

Show what voice ports router is equipped with:

```
show voice port summary
```

- Each `ephone-dn` shows up as an EXFS port.

### Configuring Analog Voice Ports

##### FXS (Foreign Exchange Station) Ports
- Connect to end-stations: typical analog devices such as telephones, fax machines, modems.
- Three common areas of configuration:
    - Signaling (`signal`):
        - **Loop start**: default; typically used for connecting to analog devices.
        - **Ground start**: needs to be configured; typically for connecting to PBX equipment.
    - Call progress tones (`cptone`): default is US
    - Call ID information (`station id`)

```
voice-port 0/2/0
  signal <signaling method>

  cptone ?                      # Displays call progress tone countries
  cptone <country code>         # Sets call progress tone to country

  station id name <name/description>
  station id number <number>
```

##### FXO (Foreign Exchange Office) Ports

- Acts as a trunk to the PSTN central office (CO) or PBX systems.
- Use same commands as FXS ports such as `signal` and `station-id`.
- Additional commands: `dial-type`, `ring number`

```
voice-port 0/3/0
  dial-type < dtmf / pulse >
  ring number <number>          # Number of rings before router answers incoming call to the FXO port (default = 1)
```

### Configuring Digital Voice Ports

Cisco provides digital T1 and E1 ports in the form of voice and WAN interface cards (VWICs) for routers).

Two types of voice network configurations:

- `ds0-group` for T1/E1 channel associated signalling (CAS).
- `pri-group` for T1/E1 common channel signalling (CCS) connections (commonly referred to as ISDN Primary Rate Interface [PRI]).

Display configuration for T1:

```
show controllers t1
```

Example configuration of a T1 CAS PSTN Interface:

```
configure terminal
controller t1 1/0
 framing esf        !! esf or sf
 linecode b8zs      !! b8zs is encoding common in US
 clock source line  !! free-running | internal | line
 ds0-group 1 timeslots 1-24 type ?
```

Example configuration of a T1 CCS PSTN Interface:

```
isdn switch-type primary-5ess
controller t1 1/0
 pri-group timeslots 1-24

```

Verify configuration with `show voice port summary`

### Understanding and Configuring Dial Peers

- **Dial peers**: similar to static routes but for the voice network - by default, the CME router only knows how to reach the `ephone dn`s you configure for Cisco IP phones.
    - Define voice reachability information, i.e. phone numbers you can dial.
    - Using a dial peer, you can assign one or more phone numbers to a device.
    - Allows wildcards to define ranges of phone numbers.
- One can connect the CME router to any number of FXS, FXO, or digital T1/E1 connections.
- Internal dialing is through `ephone`, external dialing is through *dial-peers*.

Two primary types of dial peers:

- **Plain old telephone service (POTS) dial peer**: defines voice reachability information for any traditional voice connection.
- **Voice over IP (VoIP) dial peer**: voice reachability information for any VoIP connection.

### Voice Call Legs

- A call leg represents a connection to or from a voice gateway from a POTS or VoIP source.
- Each call leg represents a dial peer that must exist on the router; these dial peers define not only the reachability information (phone numbers) for the devices, but also the path the audio must travel.
- Call legs are matched on both the inbound and outbound directions; dial peers must be configured to match voice traffic in both inbound and outbound directions.

### Configuring POTS Dial Peers

- Defines reachability information for anything from traditional telephony: devices connected to FXO, FXS, E&M, and digital BRI/T1/E1. These connected devices do not have an IP address.

```
dial-peer voice <number-tag> pots
 destination-pattern <prefix or full telephone number>
 port 0/0/0
 exit
```

Verifying dial-peers:

```
show dial-peer voice
show dial-peer voice summary
```

Debugging dial-peers:

```
debug voip dialpeer
```

Configuring a POTS Dial Peer for a T1 interface

```
dial-peer voice 2000 pots
 destination-pattern 2...
 no digit-strip
 port 1/0:23
```

- With POTS dial peers, the router automatically strips any explicitly defined digits before forwarding the call. E.g. 9....... => 9 is stripped before forwarding the remaining digits to the PSTN.
- VoIP dial peers do not automatically strip digits.

##### Dealing with automatic digit-stripping of POTS dial peers

Example of configuration of a North American PSTN dial plan on a router, where the T1 CAS voice port 1/0:1 is connected to the PSTN, and internal users must dial 9 for outside PSTN access.

```
dial-peer voice 90 pots
 description Service dialing
 destination-pattern 9[469]11
 forward-digits 3
 port 1/0:1
 exit

dial-peer voice 91 pots
 description 10-digit dialing
 destination-pattern 9[2-9]..[2..9]......
 port 1/0:1
 exit

dial-peer voice 92 pots
 description 11-digit dialing
 destination-pattern 91[2-9]..[2-9]......
 forward-digits 11
 port 1/0:1
 exit

dial-peer voice 93 pots
 description International dialing
 destination-pattern 9011T
 prefix 011
 port 1/0:1
 exit
```

- `forward-digits <number>` specifies number of right-justified digits to forward.
- `prefix <number>` adds specified digits before forwarding.

### Configuring VoIP Dial Peers

Example:

```
dial-peer voice 2000 voip
 destination-pattern 2...
 session target ipv4:<ip address of next-hop>
 codec g711ulaw         ! Optional; default is G.729

```

- Note that calls will fail if there is a **codec mismatch** - when codec values do not match between two routers.

### Using Dial Peer Wildcards

```
.   Period      Matches any dialed digit or the * key on telephone keypad
+   Plus        Matches one or more instances of the preceding digit
[]  Brackets    Matches a range of digits; ^ before a digit designates "do not match"
T   Any         Matches any number of dialed digits (from 0-32)
,   Comma       Inserts a 1-second pause between dialed digits.
```

- To match a destination pattern of any using T, the destination should be created as `.T`,

See P.134 Table 6-4 and 6-5 for example bracket wildcards / PSTN destination patterns for North America.

### Private Line Automatic Ringdown (PLAR)

- PLAR configurations rely heavily on existing dial peers to complete a call.
- Ports configured with PLAR capabilities automatically dial a number as soon as the port detects an off-hook signal.
- Configuring PLAR connections are useful for *incoming calls for analog FXO trunks*

Example to hard-code FXS voice port 0/0/0 to dial 1102 when the user lifts the handset:

```
voice-port 0/0/0
 connection plar 1102

 connection plar opx 1101
```

- With `plar opx`, voice port does not answer until the remote side has answered. If you are transferring to an extension over which you have no control (an off premise extension), it's beneficial for the port to wait to answer the call until it knows the far end is going to answer. This keeps the gateway from answering a call and then having nowhere to route it. ([Source](https://supportforums.cisco.com/discussion/10072041/connection-plar-vs-connection-plar-opx))

### Understanding Router Call Processing and Digit Manipulation

Two primary rules:

- The most specific destination pattern always wins.
- When a match is found, the router immediately processes the call.

To test which dial peer will match a specific string:

```
show dialplan number <number to test>
```

### Matching Inbound and Outbound Dial Peers

When a router receives a voice call, it must always match a dial peer in some way for the router to process the call.

A router matches inbound dial peers through the following five methods:

1. Match the dialed number (DNIS) using the `incoming called-number` dial-peer config.
2. Match the caller ID info (ANI) using the `answer-address` dial-peer config.
3. Match the caller ID info (ANI) using the `destination-pattern` dial-peer config.
4. Match an incoming POTS dial peer using the `port dial-peer configuration` command.
5. If no match has been found using the previous four methods, use *dial peer 0*.

**Dial peer 0** is like a default gateway dial peer that appears when there is no dial peer match (only for inbound dial peers, not for outbound dial peers).

### Failover Implementation

Using the `preference` command allows the router to determine which dial peer to use when destination patterns are identical.
```
dial-peer voice 10 voip
 destination-pattern 6...
 ! ...
 preference 0
 exit
dial-peer voice 11 pots
 destination-pattern 6...
 ! ...
 preference 1
 exit
```

### Transforming Dialed Numbers

Using `num-exp` global configuration command universally transforms the dialed number 0 (anywhere in an organization) to 5000, then would search for a dial peer allowing it to reach 5000.

```
voice-port 1/0/1
 connection plar 0
 exit
num-exp 0 5000
```

### Specific POTS Lines for Emergency Calls

Configure a dedicated FXO port for one local PSTN connection for emergency calling:

```
dial-peer voice 10 pots
 destination-pattern 911
 port 1/0/0
 no digit-strip
 exit

! Second dial-peer for those accustomed to dialing 9 before 911
dial-peer voice 11 pots
 destination-pattern 9911
 port 1/0/0
 forward-digits 3
```

### Using CCP to Configure a CME Dial Plan

# Understanding and Implementing CME Class of Restriction (COR)

Calling restrictions on users of the VoIP network are carried out through:

- **Partitions & Calling Search Spaces (CCS)** when using the full CUCM platform.
- **Class of Restriction** lists (incoming and outcoming) in the CME environment.

Steps in COR List Implementations

1. Define the COR tags to be used for restrictions.
2. Create the outbound COR lists.
3. Create the inbound COR lists.
4. Assign the outbound COR lists.
5. Assign the inbound COR lists.

Example:

```
configure terminal
dial-peer cor custom
 name 911
 name LOCAL
 name LD
exit
```

Create Outgoing COR Lists:

- "For this COR list to allow the call, the caller must be assigned the *member xxx* tag."

```
dial-peer cor list 911-CALL
 member 911
 exit
dial-peer cor list LOCAL-CALL
 member LOCAL
 exit
dial-peer cor list LD-CALL
 member LD
 exit
```

Create Incoming COR Lists:

- "Anyone assigned to this COR list can call dial peers requiring the *member xxx* tag."
- "I will assign these *member xxx tags* to a caller, which grant the ability to place a call."

```
dial-peer cor list 911-ONLY
 member 911
 exit
dial-peer cor list 911-LOCAL
 member 911
 member LOCAL
 exit
dial-peer cor list 911-LOCAL-LD
 member 911
 member LOCAL
 member LD
 exit
```

Assigning Outbound and Inbound COR Lists:

```
dial-peer voice 10 pots
 corlist outgoing 911-CALL
 exit
dial-peer voice 11 pots
 corlist outgoing LOCAL-CALL
 exit
dial-peer voice 12 pots
 corlist outgoing LD-CALL
 exit
ephone-dn1
 corlist incoming 911-ONLY
 exit
ephone-dn2
 corlist incoming 911-LOCAL
 exit
ephone-dn3
 corlist incoming 911-LOCAL-LD
 exit
```

# Quality of Service

- Definition of QoS: the ability of the network to provide better or special service to a set of users and applications at the expense of other users and applications.
- Voice traffic must have priority over data traffic for VoIP network operations.

### Using Cisco AutoQoS

- Deploys a template QoS configuration in line with Cisco QoS best practices, based on the bandwidth and encapsulation you configured under each of your router or switch interfaces.
- Uses CDP to detect Cisco IP phones on Cisco switches and properly configure QoS settings.
- Uses LLQ

Multiple advantages of AutoQoS to manual QoS configuration:

- Reduces time of deployment
- Provides configuration consistency
- Reduces deployment cost
- Allows manual tuning

A **trust boundary** must first be established before deployment of AutoQoS.

- A trust boundary is the point of the tnetwork where you begin trusting that the network traffic is accurately identified with the correct QoS marking.
- Cisco IP phones have the ability to mark their own traffic as high priority and strip any high priority markings from traffic sent by the attached PC.

##### Configuring AutoQoS

- Identify interface to which applying QoS makes sense: `interface <interface-id>`
- For router serial interfaces, ensure correct bandwidth statement is entered, as the router cannot auto-detect the actual speed of a WAN connection.

```
auto qos ?
```

Enable trust boundary if CDP detects a Cisco IP phone or Cisco IP Communicator (Switch):

```
auto qos voip cisco-phone
auto qos voip cisco-softphone
```

Trust markings from interface no matter what device (e.g. router - switch); does not rely on CDP:

```
auto qos voip trust
```

Use AutoQoS without trusting any existing markings on packets; router would re-mark all traffic using access lists or Network-Based Application Recognition (NBAR) to identify traffic (higher processor-utilization tasks):

```
auto qos voip     # Router or Layer 3 switch
```

CoS: Class of Service - marking that exists in the layer 2 header of a frame.
ToS: Type of Service - marking that exists in the layer 3 header of a packet.

### Configuring Manual Local Directory Entries

```
telephony-service
 directory ?

 directory first-name-last
 directory last-name-first

 directory entry <directory-entry-tag> <ephone-dn> name <dir-name>
```

### Forwarding Calls

```
ephone-dn <tag>
 call-forward all               ! forward all calls
 call-forward busy              ! forward calls on busy
 call-forward max-length        ! num digits allowed for CFwdAll
 call-forward night-service     ! on activated night service
 call-forward noan              ! no answer
```

### Configuring Call Transfers

```
telephony-service
 transfer-system ?
 transfer-system full-blind     ! call transfers without consultation
 transfer-system full-consult   ! H.4502/SIP call transfers with consultation
 transfer-system local-consult  ! call transfers with local consultation

 transfer-pattern ?
 transfer-pattern 5...          ! allow transfers to 5xxx extensions
 transfer-pattern 9..........   ! allow transfers to local 10-digit PSTN nums
```

### Configuring Call Park

- Parks the caller on hold at an extension rather than on a specific line.
- Works by finding free ephone-dns that have not been assigned to an IP phone and have specifically designated as a call park slot.

```
ephone-dn <tag>
 number 3001
 name Maintenance
 park-slot
 exit
ephone-dn <tag>
 number 3002
 name Sales
 park-slot timeout 60 limit 10 recall
```

### Misc: Enabling Voice in Router 2911 (Packet Tracer)

In (config)#:

```
license boot module c2900 technology-package uck9

# ... Accept license, save config and reload

do copy run s
do reload
```
