# Show Commands: Filtering Output

Configure number of lines to display during `show`

```
terminal length <number of lines>
terminal length 0     # No pause
```

Pipe after the `show` then `<filter parameter> <filter expression>`
Can be used in combination with any `show` command.

```
show running-config | section line vty
```

Filtering parameters:

- `section`: entire section that starts with expression
- `include`: all output lines that match expression
- `exclude`: excludes all output lines that match expression
- `begin`: all output lines from a certain point, starting with the line that matches the filtering expression.

Useful examples:

```
show ip interface brief | include up
show ip interface brief | exclude unassigned
show ip route | begin Gateway
```

### Command History

- Ctrl-P or Up-Arrow
- Ctrl-N or Down-Arrow

```
show history    # (PrivExec)
```

### Change size of history buffer

```
terminal history size <num>   # (UserExec)
```

# Boot Environment

### Configure BOOT Environment Variable

```
show boot                                       #  Displays current IOS boot file
boot system flash:/<path>/<IOS_filename.bin>    # (GCM)
```

### Accessing the Boot Loader

The boot loader can be accessed through a console connection following these steps:

1. Connect a PC by console cable to the switch console port. Configure terminal emulation software to connect to the switch.
2. Unplug the switch power cord.
3. Reconnect the power cord to the switch and, within 15 seconds, press and hold down the Mode button while the System LED is still flashing green.
4. Continue pressing the Mode button until the System LED turns briefly amber and then solid green; then release the Mode button.
5. The boot loader switch: prompt appears in the terminal emulation software on the PC.

Basic directory listing in the boot loader:
```
dir flash:
```

# Cisco Software Upgrade Procedure

See: [Cisco Access Routers Software Upgrade Procedures](http://www.cisco.com/c/en/us/support/docs/routers/3800-series-integrated-services-routers/49044-sw-upgrade-proc-ram.html)

# Configure Router as TFTP Server

```
tftp-server flash:<ios_image_name.bin>
```
