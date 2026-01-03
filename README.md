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
