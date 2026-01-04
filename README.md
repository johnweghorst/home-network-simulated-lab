# Home Network Simulated Lab

Welcome! The goal of this lab in Cisco Packet Tracer is to design and simulate a segmented home network topology that separates a PC, a gaming device (will use a laptop), mobile devices, and IoT devices to improve security and manageability.

This lab will be in-depth and it's Cisco hardware oriented.  It will implement VLANs to logically isolate devices, NAT, DHCP and DNS to provide automated IP addressing and name resolution, inter-VLAN routing to allow communication between networks, and basic security configurations to restrict unnecessary access between networks. We will also be implementing an ACL to allow only the PC access to the router via SSH.

This lab will assume you’re already somewhat familiar with Packet Tracer, placing devices and connections, basic Cisco commands and how to navigate IOS. If you aren't, it's best to watch a basic tutorial on how set things devices and connections up. I won’t include every screenshot of entering commands but the commands and configurations will be shown so the lab can be reproduced. Additional configurations, such as port security and ACLs, are included to demonstrate security best practices and aren’t strictly necessary for a home network but are beneficial to implement. Some liberties are taken due to the limitations of Packet Tracer.

# Tools & Software
* Cisco Packet Tracer
* Cisco IOS 
* Windows-based end devices (PC & laptop)
* Mobile devices (smartphone/tablet)
* IoT devices (smart TV, camera, etc.)
# VLAN and IPv4 Addressing

| VLAN | Name    | Purpose                         | Subnet            | Gateway         |
|-----:|---------|---------------------------------|-------------------|-----------------|
| 10   | Trusted | PC, Laptop, iPhones, iPads      | 192.168.10.0/24   | 192.168.10.1    |
| 20   | Gaming | PS4, Gaming Laptop              | 192.168.20.0/24   | 192.168.20.1    |
| 30   | IoT  | Ring, Smart TV                  | 192.168.30.0/24   | 192.168.30.1    |
| 40   | Guest   | Guest devices                   | 192.168.40.0/24   | 192.168.40.1    |
| 99   | Management    | Router, Switch, WLC / AP        | 192.168.99.0/24   | 192.168.99.1    |
# Router Setup
We’ll start with router configuration first. Place the router in the logical topology, open it and head to the CLI.
We’ll set a hostname of HomeRouter, set passwords for the console connection and virtual line passwords, the password for privileged exec mode and SSH access first. All passwords will be cisco just for ease. Use a secure password out of a lab environment of course. We will also encrypt our passwords and while this isn’t strong encryption, it will add a layer of security and it prevents passwords from being immediately readable by prying eyes in the running config.
```
Router> enable
conf t
service password-encryption
```
Next, we’ll name the router and setup passwords for privileged exec mode, the console line and the virtual terminal lines, as well as allowing connectivity via SSH only on them. The console line is for direct access and the VTY lines are for remote access over the network. We use login local for SSH because we want it username AND password protected for security purposes since the data is going over the network. 
```
hostname HomeRouter
enable secret cisco
line con 0
password cisco
login
line vty 0 15
password cisco
login local
transport input ssh
```
Because of password-encryption, our passwords are encrypted in the running config (show run command).
<img width="975" height="295" alt="image" src="https://github.com/user-attachments/assets/1456928f-fb7f-4458-8d06-e24bd48d63e3" />
<img width="628" height="508" alt="image" src="https://github.com/user-attachments/assets/2b49473f-e388-4e4e-abb5-2ace7968b7ae" />

Exit to get back to global configuration mode.

Here, we will also set a banner message that will display a warning message if someone happens to get unwanted access to the router (via SSH, for example). While it doesn’t prevent access, it serves as a legal deterrent by clearly stating that access is restricted to authorized users only. if illegal activity was performed or your public IP address was used maliciously, a banner can help establish intent and provide notice.
```
banner motd #Unauthorized access to this system is prohibited. Activities may be monitored and logged. Disconnect immediately if you are not an authorized user.#
```
<img width="975" height="522" alt="image" src="https://github.com/user-attachments/assets/a1d81b3b-f7d5-4dc1-b755-f991bced5ea5" />
Next, will we set up the domain name that is required to generate the RSA keys for SSH. SSH requires a domain name to generate a unique cryptographic key pair. We’ll also set a username and password for logging in via SSH. We’ll also set SSH version to 2 because SSH version 1 limits defense against new threats and has numerous documented vulnerabilities.
```
ip-domain name mydomain.com
crypto key generate rsa 1024
username admin privilege 15 secret cisco
ip ssh version 2
```
We also need to set up DNS so our router can resolve domain names. We’ll use google’s public DNS server for this so we can also set up a DNS server to see how they function.

```
ip name-server 8.8.8.8
```

Here is what’s shown in the running config.

<img width="623" height="167" alt="image" src="https://github.com/user-attachments/assets/e45c3440-007a-4b7c-b6bc-8f9e5d0b4223" />



