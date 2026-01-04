# Home Network Simulated Lab in Cisco Packet Tracer

Welcome! The goal of this lab in Cisco Packet Tracer is to design and simulate a segmented home network topology that separates PCs and mobile devices, gaming devices (will use a laptop and a mock gaming console), and IoT/Guest devices to improve security and manageability.

This lab will be implementing VLANs to isolate devices, NAT, DHCP and DNS to provide automated IP addressing and name resolution, inter-VLAN routing to allow communication between networks, and basic security configurations to restrict unnecessary access between VLANs. We will also be implementing ACLs to allow only the PC access to the router via SSH and for strict policing for the IoT devices. For the Wi-Fi, we’ll be configuring a Wireless LAN Controller and Lightweight Access Point for the wireless VLAN routing. In reality for a small home network, a combo unit of a router, WLC and AP could be purchased, and a separate WLC isn’t necessary. 

However, this lab takes liberties to simulate an enterprise-style deployment to demonstrate CCNA-level knowledge, as well as work around the limits of Packet Tracer.

This lab will assume you’re already somewhat familiar with Packet Tracer, placing devices and connections, basic Cisco commands and how to navigate IOS. If you aren't, it's best to watch a basic tutorial on how set things devices and connections up. I won’t include every screenshot of entering commands but the commands and configurations will be shown so the lab can be reproduced. Additional configurations, such as port security and ACLs, are included to demonstrate security best practices and aren’t strictly necessary for a home network but are beneficial to implement. 

Some liberties are taken due to the limitations of Packet Tracer, and the scope of this lab is intended to be at CCNA-level.


# Tools & Software
* Cisco Packet Tracer
* Cisco IOS 

# Core Device list
* Cisco ISR 1941 Router
* Cisco Catalyst 2960 Switch
* Wireless LAN Controller (WLC-PT)
* Lightweight Access Point (LAP-PT)

# End Devices
* PC-PT, 2 Smartphones, and 2 Tablets (Trusted - VLAN 10)
* Laptop-PT, Generic Wireless End Device (Gaming - VLAN 20)
* Generic Wireless End Device/TV, Guest Phone  (IoT/Guest - VLAN 30)
* Server (DNS/Internet simulation purposes)

# VLAN and IPv4 Addressing

| VLAN ID | Name      | Subnet           | Gateway        | Devices / Description                            |
|---------|-----------|-----------------|----------------|-------------------------------------------------|
| 10      | Trusted  | 192.168.10.0/24 | 192.168.10.1  | PCs and mobile devices; trusted network        |
| 20      | Gaming   | 192.168.20.0/24 | 192.168.20.1  | Gaming laptops/consoles                        |
| 30      | IoT/Guest     | 192.168.30.0/24 | 192.168.30.1  | IoT devices: smart TVs, Guest phone, Ring doorbell   |
| 99      | Management    | 192.168.99.0/24 | 192.168.99.1  | Router, switch, WLC management devices        |
| 999     | Native   | n/a             | n/a            | Native VLAN for trunk ports; unused            |

The topology will be simply organized by VLANs in numerical order, and the devices sorted likewise. Make the following connections:
* Router G0/1 to Server Fa0.
* Router G0/0 to Switch G0/1.
* Switch G0/2 to WLC Gi0.
* Switch Fa0/1 to Desktop PC.
* Switch Fa0/2 to Gaming Laptop.

<img width="975" height="669" alt="image" src="https://github.com/user-attachments/assets/39e33b4f-ba0e-4d15-8be6-fda4c8b217d8" />



# Router Setup
 We’ll start with router configuration first. Open it and head to the CLI.
We’ll set a hostname of HomeRouter and cover logging in to console connection, virtual lines, privileged exec mode and SSH by local admin account only. All passwords will be cisco just for ease. Use a secure password out of a lab environment of course. We’ll also add an additional layer of security for privileged exec mode on top of our local admin account, just in case someone should happen to get in. 

While the encryption used for this isn’t strong encryption, it will add a layer of security and it prevents passwords from being immediately readable by prying eyes in the running config as well. 

The console line is for direct access and the VTY lines are for remote access over the network. We use login local because we want the username AND password we set up for better security. Enable secret is for privileged exec mode. 

Username and password will be set up during SSH configuration.
```
Router> enable
conf t
service password-encryption
hostname HomeRouter
enable secret cisco
line con 0
login local
line vty 0 4
login local
transport input ssh
line vty 5 15
login local
transport input none

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

# SSH setup 
```
ip domain name mydomain.com
crypto key generate rsa 1024
username admin privilege 15 secret cisco
ip ssh version 2
```
We also need to set up DNS so our router can resolve domain names.
```
ip name-server 203.0.113.1
```
Here is what’s shown in the running config.

<img width="598" height="156" alt="image" src="https://github.com/user-attachments/assets/08613b52-e358-469a-863b-8986737c4f7e" />

# DNS Setup

We’ll go ahead and set up the DNS server now as well. The server will have a default gateway of HomeRouter’s outside interface IP address (203.0.113.1). Open up the server and head to Desktop > IP Configuration.

<img width="975" height="981" alt="image" src="https://github.com/user-attachments/assets/8a8452ec-59ef-4ed5-bc74-d414c981556d" />

Next, we’ll add some domain names to the server for address resolution. Head to the Services tab then choose DNS. Turn it on and add some A records. An A record maps a specific IPv4 address to a domain name.

<img width="975" height="975" alt="image" src="https://github.com/user-attachments/assets/7b42682d-ad03-4827-a193-c216fa6965ee" />

That’s all for the DNS server.

# Router configuration continued 
 Back at the router, we’ll set the router’s outside IP address now as well.
 
```
int g0/1
ip add 203.0.113.1 255.255.255.252
no shut
```
We’ll ping from the router to verify connectivity.


<img width="975" height="293" alt="image" src="https://github.com/user-attachments/assets/c4d74e68-bc88-473f-839c-a9d20e445eea" />


It’s successful. We are connected to the DNS server. 

# Inter-VLAN routing

With that part done, we will set up inter-VLAN routing. It will allow our segmented networks to access the Internet while remaining logically isolated, and we will write Access Control Lists later on to limit connectivity between them. I have decided on a router-on-a-stick because there’s only a small amount of devices on the network, and there won’t be a ton of traffic getting routed at once on the trunk, so a layer 3 switch is not necessary with this network. It is best practice to change the native VLAN to an unused VLAN rather than leaving it at the default of 1 to prevent certain types of attacks.

Back at global config mode,
```
int g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

int g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

int g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0

int g0/0.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0

int g0/0.999
encapsulation dot1Q 999 native

int g0/0
no shut
```
Configurations can be checked with show ip int brief.

<img width="975" height="193" alt="image" src="https://github.com/user-attachments/assets/c91d5cee-d54d-49c0-86aa-b1558235c186" />

Next, we’ll set up our DHCP on the router. While this isn’t completely necessary in this setup, we’re going to do for future proofing any devices we may want to add in the future. Static IPs on our devices would work just as well for the non-Guest networks.

```
ip dhcp pool Trusted
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 203.0.113.2

ip dhcp pool Gaming
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 203.0.113.2

ip dhcp pool IoT
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 203.0.113.2
 

ip dhcp pool Guest
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 203.0.113.2
 ```

DHCP isn’t necessary for our management network. They will be set with static IPs because we don’t want their IP addresses to change. We can check our configuration in the running config.

<img width="752" height="623" alt="image" src="https://github.com/user-attachments/assets/eeed531a-5c2b-4dff-9c8d-585f13b5f473" />

We can check that our DHCP set up is successful by changing our Desktop PC for DNS. Open up the Desktop PC and head to Desktop > IP config and choose DHCP. Success.

<img width="975" height="494" alt="image" src="https://github.com/user-attachments/assets/b2c3391f-9009-47eb-b1b5-1745e1421dce" />

We can ping the DNS server to verify connectivity there as well.

<img width="975" height="865" alt="image" src="https://github.com/user-attachments/assets/a64c9ede-a0e5-4add-8f9d-f9a6c7cff8f4" />

# NAT & ACLs

With DHCP successfully configured, we can set up the NAT and ACLs. NAT will allow our devices to use the router’s outside global IP address for communications outside of our network. Since most ISPs will only distribute one public IP for customers, this is essential to be able to connect over the internet with all of our devices. We will implement NAT overload, known as PAT (port address translation), which will add an ephemeral port number to our router’s public IP address so all of our internal devices are able to use the same IP address for outside communication. (ex: 203.0.113.1:45678 for the Desktop PC, 203.0.113.1:43256 for a Guest phone).

These commands will identify the inside interfaces (our internal VLAN sub-interfaces where our private IPs reside) and the outside interface (the router’s public IP) to perform the translation.

