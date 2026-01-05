# Home Network Simulated Lab in Cisco Packet Tracer

Welcome! The goal of this lab in Cisco Packet Tracer is to design and simulate a segmented home network topology that separates PCs, a gaming device (will use a laptop), mobile devices, and IoT devices to improve security and manageability.

This lab will be implementing VLANs to isolate devices, NAT, DHCP and DNS to provide automated IP addressing and name resolution, inter-VLAN routing to allow communication between networks, and basic security configurations to restrict unnecessary access between VLANs. We will also be implementing ACLs to allow only the PC access to the router via SSH and for strict policing for the IoT devices. For the Wi-Fi, we’ll be configuring a Wireless LAN Controller and Lightweight Access Point for the wireless VLAN routing. In reality for a small home network, a combo unit of a router, WLC and AP could be purchased, and a separate WLC isn’t necessary. 

However, it is important to note that this lab takes liberties to simulate an enterprise-style deployment to demonstrate CCNA-level knowledge, as well as work around the limits of Packet Tracer.

This lab will assume you’re already somewhat familiar with Packet Tracer, placing devices and connections, basic Cisco commands and how to navigate IOS. If you aren't, it's best to watch a basic tutorial on how set things devices and connections up. I won’t include every screenshot of entering commands but the commands and configurations will be shown so the lab can be reproduced. Access control lists are included to demonstrate security best practices and aren’t strictly necessary for a home network but are beneficial to implement. 
Some liberties are taken due to the limitations of Packet Tracer, and the scope of this lab is intended to be at CCNA-level.

The Packet Tracer is also be included.

<img width="975" height="617" alt="image" src="https://github.com/user-attachments/assets/db96f81b-eb09-4a9f-a998-d4d9deba0dea" />


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
* Laptop-PT, Generic Wireless End Device, Wireless Gaming Laptop (Gaming - VLAN 20)
* Generic Wireless End Device/TV/Guest Phone  (IoT/Guest - VLAN 30)
* Server (DNS/Internet simulation purposes)

The topology will be simply organized by VLANs in numerical order, and the devices sorted likewise. Make the following connections:
* Router G0/1 to Server Fa0.
* Router G0/0 to Switch G0/1.
* Switch G0/2 to WLC Gi0.
* Switch Fa0/1 to Desktop PC.
* Switch Fa0/2 to Gaming Laptop. 
* Switch Fa0/3 to LAP Gi1.

# VLAN and IPv4 Addressing

| VLAN ID | Name      | Subnet           | Gateway        | Devices / Description                            |
|---------|-----------|-----------------|----------------|-------------------------------------------------|
| 10      | Trusted  | 192.168.10.0/24 | 192.168.10.1  | PCs and mobile devices; trusted network        |
| 20      | Gaming   | 192.168.20.0/24 | 192.168.20.1  | Gaming laptops/consoles                        |
| 30      | IoT/Guest     | 192.168.30.0/24 | 192.168.30.1  | IoT devices: smart TVs, Guest phone, Ring doorbell   |
| 99      | Management    | 192.168.99.0/24 | 192.168.99.1  | Router, switch, WLC management devices        |
| 999     | Native   | n/a             | n/a            | Native VLAN for trunk ports; unused            |



# Router setup

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

<img width="975" height="286" alt="image" src="https://github.com/user-attachments/assets/3fcbfab7-e077-48c2-801c-d2387bf477c7" />
<img width="477" height="411" alt="image" src="https://github.com/user-attachments/assets/a16b97f3-0b25-4206-a703-44fa7a297113" />

Exit to get back to global configuration mode.
Here, we will also set a banner message that will display a warning message if someone happens to get unwanted access to the router (via SSH, for example). While it doesn’t prevent access, it serves as a legal deterrent by clearly stating that access is restricted to authorized users only. if illegal activity was performed or your public IP address was used maliciously, a banner can help establish intent and provide notice.

```
banner motd #Unauthorized access to this system is prohibited. Activities may be monitored and logged. Disconnect immediately if you are not an authorized user.#
```

<img width="975" height="522" alt="image" src="https://github.com/user-attachments/assets/7941394c-2f4f-4a8f-b23f-9d7b2f422a33" />

Next, will we set up the domain name that is required to generate the RSA keys for SSH. SSH requires a domain name to generate a unique cryptographic key pair. We’ll also set a username and password for logging in via SSH. We’ll also set SSH version to 2 because SSH version 1 limits defense against new threats and has numerous documented vulnerabilities.

# SSH setup 
From global config mode:

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

<img width="598" height="156" alt="image" src="https://github.com/user-attachments/assets/ccbc52af-e5f8-4068-90d1-c9a568f3f4d4" />

We’ll go ahead and set up the DNS server now as well. The server will have a default gateway of HomeRouter’s outside interface IP address (203.0.113.1). Open up the server and head to Desktop > IP Configuration.

<img width="975" height="981" alt="image" src="https://github.com/user-attachments/assets/81b8faea-378e-49eb-a334-0c7ec4136ce3" />

Next, we’ll add some domain names to the server for address resolution. Head to the Services tab then choose DNS. Turn it on and add some A records. An A record maps a specific IPv4 address to a domain name.

<img width="975" height="975" alt="image" src="https://github.com/user-attachments/assets/6a8b56ec-9d97-47c4-b290-5bdac293c27e" />

That’s all for the DNS server. We’ll set up the switch now so we can start using ping to verify connectivity.

# Switch configuration
Perform the same basic set up steps on the switch as we did the router. After, we’ll configure the VLANs and the trunks on the switch, one to the router and one to our WLC, then add our wired connections to their respective VLANs. We will also set up the switch’s SVI (switched virtual interface) so we can connect with SSH over the network. This will require adding a default gateway to the router as well. It is best practice to change the native VLAN to an unused VLAN. For the WLC, the native vlan must be the management VLAN. SSH configuration will be shown again for redundancy and because it is a crucial step. From global config mode:

```
vlan 10
 name Trusted
vlan 20
 name Gaming
vlan 30
 name Untrusted
vlan 99
 name Management
vlan 999
 name Native
```

<img width="955" height="527" alt="image" src="https://github.com/user-attachments/assets/62131e3e-1995-453f-aa81-f00f0e1cadad" />

```
int g0/1
 description Trunk to Router
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,99,999

int g0/2
 description Trunk to WLC / AP
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,99,999

int f0/1
 switchport mode access
 switchport access vlan 10

int f0/2
switchport mode access
switchport access vlan 20

int f0/3
switchport mode access
switchport access vlan 99
```

Verify our trunk ports with show int trunk:

<img width="975" height="420" alt="image" src="https://github.com/user-attachments/assets/385d70ea-df62-4b70-9ea9-cf3a29aa1107" />

And the running config:

<img width="864" height="188" alt="image" src="https://github.com/user-attachments/assets/9f496600-933b-41c6-a0a8-4cbb8f9d91a5" />

We’ll also test our connection to the default gateway with ping.

<img width="975" height="170" alt="image" src="https://github.com/user-attachments/assets/902113b5-c725-44b4-928c-24fc6426c8a3" />

Successful! Our switch is correctly configured. We can view the VLANs with show the vlan brief command to verify our configuration is correct.

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/9a949d31-fcfe-4953-9c9c-c63b0b3efe2c" />

# SSH setup 
Make sure the switch has been named as this is necessary for SSH configuration. From global config mode:

```
ip domain name mydomain.com
crypto key generate rsa 1024
username admin privilege 15 secret cisco
ip ssh version 2
line vty 0 4
login local
transport input ssh
int vlan 99
ip add 192.168.99.2 255.255.255.0
no shut
exit
ip default-gateway 192.168.99.1
```

# Router configuration continued 
Back at the router, we’ll set the router’s outside IP address now as well.
From global config mode:

```
int g0/1
ip add 203.0.113.1 255.255.255.252
no shut
```

We’ll ping from the router to verify connectivity.

<img width="975" height="293" alt="image" src="https://github.com/user-attachments/assets/68e86f6e-c516-4497-be00-1f6ecd599d18" />

It’s successful. We are connected to the DNS server. 

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

<img width="975" height="193" alt="image" src="https://github.com/user-attachments/assets/463e680e-5aad-49bf-8f4a-a5f123fc571c" />

Next, we’ll set up our DHCP on the router. While this isn’t completely necessary in this setup, we’re going to do for future proofing any devices we may want to add in the future. Static IPs on our devices would work just as well for the non-Guest networks.
From global config mode:
```
ip dhcp excluded-address 192.168.99.1 192.168.99.3

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

ip dhcp pool Management
 network 192.168.99.0 255.255.255.0
 default-router 192.168.99.1
 dns-server 203.0.113.2
 ```

DHCP is configured for our management network so our LAP can get an IP address automatically and recognize the WLC. This also leaves the option open for adding more LAPs in the future.

Our router, switch, and WLC will be configured with static IPs (192.168.99.1, .2, and .3 respectfully) because we don’t want their IP addresses to change. We can check our configuration in the running config. For the excluded IPs, .2 isn’t show because it’s configured as a range of excluded IPs.

<img width="975" height="783" alt="image" src="https://github.com/user-attachments/assets/d5b937dd-bfb6-4a6b-b535-1303913e473d" />

We can check that our DHCP set up is successful by changing our Desktop PC for DNS. Open up the Desktop PC and head to Desktop > IP config and choose DHCP. Success.

<img width="975" height="494" alt="image" src="https://github.com/user-attachments/assets/61212bc3-bc7a-4d43-b14b-39f99387e1ee" />

We can ping the DNS server to verify connectivity there as well.

<img width="975" height="865" alt="image" src="https://github.com/user-attachments/assets/ceeb17d9-5b17-41cf-9eb5-945d6065ee15" />

# NAT And ACL set up

With DHCP successfully configured, we can set up the NAT and ACLs. NAT will allow our devices to use the router’s outside global IP address for communications outside of our network. Since most ISPs will only distribute one public IP for customers, this is essential to be able to connect over the internet with all of our devices. We will implement NAT overload, known as PAT (port address translation), which will add an ephemeral port number to our router’s public IP address so all of our internal devices are able to use the same IP address for outside communication. (ex: 203.0.113.1:45678 for the Desktop PC, 203.0.113.1:43256 for a Guest phone).

These commands will identify the inside interfaces (our internal VLAN sub-interfaces where our private IPs reside) and the outside interface (the router’s public IP) to perform the translation.
From global config mode:

```
 int g0/1
 ip nat outside

int g0/0.10
 ip nat inside

int g0/0.20
 ip nat inside

int g0/0.30
 ip nat inside
```
Management VLAN is not needed for NAT as we won’t be connecting to the internet. 

Double check our configurations in the running config:

<img width="892" height="1058" alt="image" src="https://github.com/user-attachments/assets/90eb5213-4213-415b-ad7e-03244f51e82b" />

# NAT overload (PAT) configuration

NAT configuration works with access lists. We’ll set up WHICH networks we want translated with it. This can be done with either a standard numbered or standard named ACL. We’ll choose named because this allows us to make easier configurations to it if needed in the future. We will also configure the router’s NAT source list and the interface that the addresses will get translated to. The last line is what allows our inside addresses to get translated to the router’s public IP. It identifies the source list and the interface of the public IP.
From global config mode:

```
ip access-list standard Nat_Inside
permit 192.168.10.0 0.0.0.255
permit 192.168.20.0 0.0.0.255
permit 192.168.30.0 0.0.0.255
exit
ip nat inside source list Nat_Inside interface g0/1 overload
```
Check our configurations with the show access-lists command and the running config.

<img width="807" height="583" alt="image" src="https://github.com/user-attachments/assets/233a3ba7-6b44-4828-b508-a8758aa0541d" />
<img width="975" height="249" alt="image" src="https://github.com/user-attachments/assets/78d0f3f6-7188-42d4-b944-b439fd5a64ad" />

 

Normally, we can see the NAT translation in action but due to how I’ve designed this network, no translations are occurring since everything is a directly connected route in the router’s routing table. But in a true network, the commands

```
Show ip nat statistics *
```

And 

```
Show ip nat translations
```

Will show the translated internal addresses as the router’s public IP address.

# Access Control Lists

This batch of ACLs are what’s going to allow us to actually segment our network and are the backbone of why I designed my network this way. The Trusted network can access everything, Gaming and IoT/Guest networks get internet access only and can’t connect to our other networks. Only devices in the Trusted network can access the management devices via SSH.  We’ll also need to add special permissions for DHCP, since when devices don’t have an IP address and it sends DHCP Discover messages, its address is 0.0.0.0. Likewise, the DHCP server response broadcasts to 255.255.255.255. ACLs are applied inbound on our ports to stop unwanted traffic as early as possible. I won’t discuss wildcard masks heavily here as they are very customizable, but the basic way to implement them is they are the opposite of the subnet mask for the network you’re working with.
Trusted VLAN ACL:
```
ip access-list extended Trusted_In
permit udp any any eq bootps
permit udp any any eq bootpc
permit ip 192.168.10.0 0.0.0.255 any
int g0/0.10
ip access-group Trusted_In in
```

Gaming ACL:
```
ip access-list extended Gaming_In
permit udp any any eq bootps
permit udp any any eq bootpc
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
deny ip 192.168.20.0 0.0.0.255 192.168.99.0 0.0.0.255
permit ip 192.168.20.0 0.0.0.255 anyint g0/0.20
ip access-group Gaming_In in
```
Guest/IoT ACL

```
ip access-list extended IoT_Guest_In
permit udp any any eq bootps
permit udp any any eq bootpc
deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
permit ip 192.168.30.0 0.0.0.255 anyinterface g0/0.30
 ip access-group IoT_Guest_In in
```

Configure the following on both the router and switch:

```
ip access-list extended Mgmt_Only
permit ip 192.168.10.0 0.0.0.255 any
line vty 0 4
 access-class Mgmt_Only in
```

Once again we’ll check our configurations are right with show access-lists.

<img width="975" height="635" alt="image" src="https://github.com/user-attachments/assets/20330755-f9ad-43fa-9608-d7fa0f81e2ad" />

We can also use a modified show run command to check that our ACLs were applied successfully.

```
show run | include interface|ip access-group
```
 
And the VTY lines:

<img width="564" height="291" alt="image" src="https://github.com/user-attachments/assets/c1521687-01a9-4be9-952a-36e33b243cae" />

# Testing connectivity

With our network successfully configured, we can test that our ACLs are working correctly. 

Our trusted Desktop PC can’t reach our Gaming Laptop in the Gaming VLAN as configured:

<img width="956" height="655" alt="image" src="https://github.com/user-attachments/assets/61897843-eebb-444b-82c9-6eee3847ec17" />

Our gaming laptop gets destination host unreachable if we try to ping the Desktop PC in VLAN 10 because of the ACL we configured, which drops the packet directly at the router. It also successfully got its IP address, default gateway, and DNS server from the DHCP server. It is unable to connect to our Desktop PC in the Trusted network as we configured. 

<img width="975" height="360" alt="image" src="https://github.com/user-attachments/assets/2b03b936-70c3-4185-9c28-ef16b98184f7" />

<img width="611" height="481" alt="image" src="https://github.com/user-attachments/assets/ccccb0e4-db7c-4fb9-8286-953159e7e775" />


Both devices can successfully ping the simulated internet, which is our DNS server.

<img width="975" height="696" alt="image" src="https://github.com/user-attachments/assets/88f8cf30-915b-45ed-ae32-bc14e4239bf9" />
<img width="975" height="453" alt="image" src="https://github.com/user-attachments/assets/e6512359-aa13-4412-b3f9-85103fbdacb9" />

We can also test our SSH to the router and switch from our trusted Desktop PC.

<img width="975" height="594" alt="image" src="https://github.com/user-attachments/assets/68428877-eb30-4468-86a9-cb42c1c77cde" />
<img width="669" height="617" alt="image" src="https://github.com/user-attachments/assets/9f26f98c-953a-4c10-9b9d-81e4c0cec84b" />

SSH is NOT reachable by our untrusted Gaming Laptop in VLAN 20 because of the ACL we configured. Nice.

<img width="850" height="791" alt="image" src="https://github.com/user-attachments/assets/5810b83a-e0a1-452a-8293-44fa15762ab4" />

# WCL Configuration

With the router and switch configurations completed, all that’s left is the WCL. I’ll be using a WLC here because it’s what’s available in Packet Tracer. In real life, I would be using a combo unit like a Ubiquiti Dream Machine.

First up, turn the LAP on. 

<img width="592" height="919" alt="image" src="https://github.com/user-attachments/assets/e0e50a13-7e29-42f0-a61c-d2df7851fedf" />

Then head to the config section of the WLC.

<img width="975" height="988" alt="image" src="https://github.com/user-attachments/assets/00057090-2244-4f89-8f77-5cf69e0c9d6d" />

Now after setting up the IP, we can log in via the web browser of our Desktop PC.

Head to http://192.168.99.3. We must create an admin account first before being able to log in for WLC configuration. I’ve made the username admin and the password C1sco1. Keep the Management VLAN ID to 0 and create a management network. 

<img width="975" height="994" alt="image" src="https://github.com/user-attachments/assets/fa91aa48-c021-46d2-90ef-1cf55deeb86d" />

***SPECIAL NOTE*** In Packet Tracer, if you set the management VLAN to the correct ID (99 in this case), and change the switch’s trunk to the WLC back to Native VLAN 999 for best practice, it will not work and we have to have it set at VLAN 0, which Packet Tracer treats as untagged traffic for the WLC management interface. This seems to be a Packet Tracer issue and is not best practice in the real world. In reality we should have the Management VLAN ID as 99 and set the native VLAN on the switch end of the trunk to a “black hole” VLAN (999 in our case).

We’ll create a quick management network here and create our other Wi-Fi networks in the proper GUI.

<img width="975" height="823" alt="image" src="https://github.com/user-attachments/assets/744967c9-5911-446e-baa7-48ba405b178e" />

We’ll create a quick management network here and create our other Wi-Fi networks in the proper GUI.

<img width="975" height="823" alt="image" src="https://github.com/user-attachments/assets/4959021c-2ec5-4cc3-ba10-119e11505804" />

Once on the “Saving the configuration…” screen, press the Fast Forward button at the bottom left corner a bunch of times. This is a known issue and limitation with Packet Tracer in that it might get hung up here during WLC setup.

<img width="358" height="89" alt="image" src="https://github.com/user-attachments/assets/e8f46779-04f6-4451-b8c8-fa7c8527acc3" />
<img width="595" height="402" alt="image" src="https://github.com/user-attachments/assets/d965b2e2-1c4b-4e0f-9f20-83e28a8270c3" />

This screen will likely hang. Just exit out and head to https://192.168.99.3 and you should be greeted with this screen.

<img width="975" height="1026" alt="image" src="https://github.com/user-attachments/assets/cab50600-7d20-4938-8aee-59774c8f320c" />

Log in and we can create are greeted by the management screen.

<img width="975" height="905" alt="image" src="https://github.com/user-attachments/assets/cbfd77af-c381-40f4-8e40-32632c5ffed6" />

As we can see, our LAP is recognized at the bottom. We can click on the details for it and see it has correctly received a management IP address from our DHCP server because we put the port it’s connected to in the management VLAN.

<img width="849" height="544" alt="image" src="https://github.com/user-attachments/assets/05cfc870-d5e6-422f-815f-adb3b1cd12db" />

Now we can create our wireless networks.

Head to Controller at the top, then “Interfaces” in the left bar.

Click “New” in the top right.

<img width="295" height="328" alt="image" src="https://github.com/user-attachments/assets/34719dfb-a2a8-4bf9-b202-ec60b8e2fc3b" />

We’ll do the Gaming VLAN first.

<img width="888" height="278" alt="image" src="https://github.com/user-attachments/assets/4cbb30ee-32fe-447d-bfd8-35db6bf31ca0" />

Click Apply.

Put in a “port number” of 1 since this is where our physical switch port is connected.
We’ll set the IP address to a range of our gaming subnet, 192.168.20.254. /24 or 255.255.255.0 subnet mask, and 192.168.20.1 as the default gateway. The DHCP server will be set to the WLC’s static IP because of limitations with Packet Tracer.

<img width="962" height="1350" alt="image" src="https://github.com/user-attachments/assets/cb4a556a-f4f3-4451-bea5-536aa1dd5804" />

Hit Apply in the top right, and it’s configured.


Next, head to the WLANs tab and click Go next to Create New.

<img width="975" height="171" alt="image" src="https://github.com/user-attachments/assets/aa39d5aa-ab33-437d-a84d-d8a72ca0edff" />

Type in a Profile name and choose the SSID. Choose any ID you want. Click Apply: 

<img width="975" height="256" alt="image" src="https://github.com/user-attachments/assets/a3f7a192-98da-412a-98d5-c5c7a832231a" />

Once inside the Gaming WLAN, click Enabled and choose the Gaming interface we just created.

<img width="975" height="841" alt="image" src="https://github.com/user-attachments/assets/b97990d2-2b9f-4d74-a285-e14852fa38b2" />

We will also set up the WPA2-PSK here. Click on Security then choose WPA+WPA2 from the dropdown. Choose your own password, then apply.

<img width="856" height="1056" alt="image" src="https://github.com/user-attachments/assets/d9a3229f-f510-4fbf-859d-783ce54171c5" />

***A note about the DHCP server. Instead of using the router’s DHCP pool like we configured, Packet Tracer will not correctly assign DHCP clients addresses from there. It will give addresses from the management domain. This is not how a proper WLC that’s correctly configured like we have done should function. However, we can set up a DHCP scope on the WLC for our wireless devices on the WLC. This will allow us to get the correct IPs from our configured DHCP pool addresses, but sometimes they will still get overridden by this bug unless the DHCP pool from the router is removed. We don’t want to do this because our Gaming Laptop is plugged in physically to the switch. This is ***NOT*** how it should function on real hardware and is a limitation or bug with Packet Tracer.***

Head to Controller > Internal DHCP Server > DHCP Scope. Enter the configurations like so:

<img width="975" height="536" alt="image" src="https://github.com/user-attachments/assets/0c22cd5f-f681-425d-b34a-805a8ea163f2" />

With that finished, we’ll have to set the Wireless settings for our Wireless Gaming Laptop. All wireless devices are configured in the same way in Packet Tracer.

Open the Wireless Gaming Laptop, then head to Configure. Choose Wireless.

<img width="975" height="820" alt="image" src="https://github.com/user-attachments/assets/2b286fe9-c557-43ef-ad44-37c9cfedb0db" />

Now, we can ipconfig /release and ipconfig /renew on our Wireless Gaming Laptop.

<img width="975" height="461" alt="image" src="https://github.com/user-attachments/assets/c874d3b2-9fad-462b-b255-369db981842c" />

And the ***BUG*** in action, for complete clarity:

<img width="975" height="503" alt="image" src="https://github.com/user-attachments/assets/e1e309cb-54db-4e64-b912-f17642cc3eb3" />

We’ll now set up our IoT/Guest network the same way, then this lab will be complete.

Follow the same steps in mapping an interface, setting up a WLAN, and setting up the DHCP scope for the IoT/Guest network. VLAN ID is 30.

<img width="861" height="325" alt="image" src="https://github.com/user-attachments/assets/b3582c3b-8f6a-4e3e-bfdd-f037fd3e0f9a" />
<img width="896" height="1350" alt="image" src="https://github.com/user-attachments/assets/694454ef-e8e8-4518-a716-558d48fba1cf" />

WLANs tab, Create New:

<img width="975" height="138" alt="image" src="https://github.com/user-attachments/assets/362040b6-ba1f-45cb-b85a-dc617fac53cd" />

<img width="975" height="840" alt="image" src="https://github.com/user-attachments/assets/17848198-abbb-40e2-a0c1-65283e8f7b35" />

And set up the PSK:

<img width="836" height="950" alt="image" src="https://github.com/user-attachments/assets/ca25af62-1768-4f3e-bef4-4fdf4551df61" />

Last step is to create a DHCP scope for this network.

<img width="914" height="959" alt="image" src="https://github.com/user-attachments/assets/32150065-6fde-4788-96a8-7d880cad1591" />

Our Guest phone successfully got the appropriate IP address from our WLC:

<img width="634" height="542" alt="image" src="https://github.com/user-attachments/assets/3d358a86-663a-4beb-a468-a4b7c8528dfc" />

We do the same for our Trusted network, but I won’t go over the steps again.

Thank you very much for reading this lab. It was a doozy figuring out how to make this topology work in Packet Tracer, but I accomplished my goal of how I would set up a home network within the limitations of Packet Tracer. It also resolidified the knowledge I obtained studying for and obtaining my CCNA certificate. 

In reality, I would just purchase an all-in-one unit that can perform all of these functions in one like a Ubiquiti Dream Machine or a pfSense firewall. Those labs may come next. Thank you for reading.



































