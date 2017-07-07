### Privilege Level Control

By default, the Cisco IOS software CLI has two levels of access to commands:

- User EXEC mode (privilege level 1) - Provides the lowest EXEC mode user privileges and allows only user-level commands available at the router> prompt.
- Privileged EXEC mode (privilege level 15) - Includes all enable-level commands at the router# prompt.

There are 16 privilege levels in total, as shown in Figure 3:

- Level 0: predefined for user-level access privileges; includes 5 commands: `disable, enable, exit, help` and `logout`.
- Level 1: default level for login with router prompt of `Router>`
- Levels 2-14: may be customized for user-level privileges.
- Level 15: Reserved for `enable` mode privileges.

```
privilege <mode> {level <level> | reset} [command]
```

### Configuring and Assigning Privilege Levels

To configure a privilege level with specific commands, use the `privilege exec level <level> [command]`.

There are two methods for assigning passwords to the different privilege levels:

- To a user that is granted a specific privilege level, use the `username <name> privilege level secret <password>` global configuration mode command
- To the privilege level, use the `enable secret level <level> <password>` global configuration mode command

```
show privilege      ! Displays current privilege

privilege <mode> {level <level> | reset} [command]
```

To configure a privilege level with specific commands:

```
privilege exec level <level> [command]
```

__Examples__:

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

### Role-Based View Configuration

There are five steps to create and manage a specific view:

__Step 1.__ Enable AAA with the `aaa new-model` global configuration mode command. Exit and enter the root view with the `enable view` or `enable view root` command.

__Step 2.__ Create a view using the `parser view <view-name>` global configuration mode command. This enables the view configuration mode. Excluding the root view, there is a maximum limit of 15 views in total.

__Step 3.__ Assign a secret password to the view using the `secret encrypted-password view` configuration mode command. Figure 2 displays the command syntax for the parser view and the secret commands.

__Step 4.__ Assign commands to the selected view using the `commands parser-mode` command in view configuration mode. Figure 3 displays the command syntax for the commands command.

__Step 5.__ Exit view configuration mode by typing the `exit` command.

Example:

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

### Configuring Role-Based CLI Superviews

There are four steps to create and manage a superview:

__Step 1.__ Create a view using the `parser view <view-name> superview` command and enter superview configuration mode.

__Step 2.__ Assign a secret password to the view using the `secret encrypted-password` command. Figure 1 displays the command syntax for the parser view superview and the secret commands.

__Step 3.__ Assign an existing view using the `view <view-name> command` in view configuration mode. Figure 2 displays the command syntax for the view command.

__Step 4.__ Exit superview configuration mode by typing the `exit` command.

To access existing views, enter the `enable view <view-name>` command in user mode and enter the password that was assigned to the custom view. Use the same command to switch from one view to another.

### Verify Role-Based CLI Views

```
enable view <view-name>
?
enable view root
show parser view all
```
