# Basic-Network-Project-VLAN-ACL-and-Password-Security
A Cisco Packet Tracer network project with 3 VLANs (Accounts, School, Head-office), router-based inter-VLAN communication, an ACL that blocks ping to Head-office, and password protection on all devices.


# Basic Network Project
A small office network built in **Cisco Packet Tracer** with three departments: Accounts, School Department and Head-office.

## What This Project Does

- Keeps each department separate using its own **VLAN** and switch.
- Connects all VLANs through one router, so every department can reach the others.
- Uses an **ACL** so Accounts and School cannot ping Head-office, while Head-office can ping everyone.
- Protects the router and all switches with **passwords**.

## Network Plan

| Department | VLAN | Network | Gateway | Router Port |
|---|---|---|---|---|
| Accounts | 10 | 192.168.10.0/24 | 192.168.10.254 | Gig0/0 |
| School Department | 20 | 192.168.20.0/24 | 192.168.20.254 | Gig0/1 |
| Head-office | 30 | 192.168.30.0/24 | 192.168.30.254 | Gig0/2 |

- Each router port connects to **Fa0/1** of its own switch.
- PCs and the printer use **Fa0/2 to Fa0/5**.
- Devices use IPs from `.1` to `.4` in their network, with mask `255.255.255.0`, and the gateway from the table above.

## Files

| File | Details |
|---|---|
| Packet Tracer file (.pkt) | In the **Releases** section on the right |
| Project report | Step-by-step report with all commands |
| Presentation | Slides about VLAN, ACL and passwords |

## How to Use

1. Download the `.pkt` file from **Releases**.
2. Open it with Cisco Packet Tracer.
3. Open a PC, go to **Desktop > Command Prompt** and run a ping test.

## Configuration Commands

### Router1

```
enable
configure terminal
hostname Router1

interface gigabitEthernet0/0
 ip address 192.168.10.254 255.255.255.0
 no shutdown
exit

interface gigabitEthernet0/1
 ip address 192.168.20.254 255.255.255.0
 no shutdown
exit

interface gigabitEthernet0/2
 ip address 192.168.30.254 255.255.255.0
 no shutdown
exit

end
write memory
```

### Switch (VLAN 10 shown)

Use `vlan 20` for School and `vlan 30` for Head-office.

```
enable
configure terminal

vlan 10
 name Accounts
exit

interface range fastEthernet0/1 - 5
 switchport mode access
 switchport access vlan 10
exit

end
write memory
```

### ACL (on Router1)

Blocks ping from Accounts and School to Head-office. Everything else is allowed.

```
configure terminal

ip access-list extended NO-PING-TO-HEADOFFICE
 deny icmp 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 echo
 deny icmp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 echo
 permit ip any any
exit

interface gigabitEthernet0/0
 ip access-group NO-PING-TO-HEADOFFICE in
exit

interface gigabitEthernet0/1
 ip access-group NO-PING-TO-HEADOFFICE in
exit

end
write memory
```

### Passwords

Use your own passwords in place of the examples.

```
enable
configure terminal

enable secret <enable-password>
service password-encryption

line console 0
 password <console-password>
 login
exit

line vty 0 4
 password <telnet-password>
 login
exit

banner motd #Authorized Access Only#

end
write memory
```

On the switches, use `line vty 0 15`.

## Test Results

| From | Ping to | Result |
|---|---|---|
| Head-office | Accounts | Success |
| Head-office | School | Success |
| Accounts | School | Success |
| School | Accounts | Success |
| Accounts | Head-office | Blocked |
| School | Head-office | Blocked |

## Troubleshooting

1. **Check the router ports.**
   ```
   show ip interface brief
   ```
   All three Gig ports must show `up / up`.

2. **Check the PC settings.** Run `ipconfig`. It needs the full IP, mask and gateway. A missing gateway is the most common mistake.

3. **Check the switch ports.**
   ```
   show vlan brief
   ```
   Fa0/1 to Fa0/5 must be in the correct VLAN.

4. **Ping from the router.** If this works, the cables, switch and VLAN are fine.
   ```
   ping 192.168.30.1
   show ip arp
   ```

5. **Check the ACL.**
   ```
   show access-lists
   ```
   The `deny` counters go up when a ping is blocked.

6. **Wait about 30 seconds** after any change, until the link lights are green. Run `write memory` so the settings stay after a reload.

## Conclusion

The router connects all three VLANs, the ACL blocks ping to Head-office, and passwords protect every device. All tests passed.

## Note

This is a lab project made for practice and learning.

## Author

**[Your Name]**
[University name] | [Course name]

## License

This project is under the [MIT License](LICENSE).
