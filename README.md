# Cisco-Enterprise-Network
In this project, I create &amp; configure an enterprise network with Cisco devices

<img src="https://i.imgur.com/288d1oQ.png"/>

In this project, we configure an entire Enterprise Cisco Network environment based on the 3-layer LAN Architecture.

<p>Environments and Technologies Used:

- Cisco Packet Tracer
- Cisco Switches
- Cisco Multilayer Switches
- Cisco Routers
- Cisco Command Line</p>

<p>For reference, pictured below is the network topology layout of the enterprise network. IP addresses of each network will be specified in the related objectives. </p>

<img src="https://i.imgur.com/288d1oQ.png"/>

<p>This Project will consist of 21 Objectives:
   
1. Setup Hostnames
2. Configure EtherChannels in each Department
3. Configure Trunk Ports
4. Configure VLANs via VTP
5. Configure Access Ports
6. Disable Unused Ports
7. Configure R1’s IP addresses
8. Enable IP Routing on all Distribution & Core switches
9. Configure PAgP EtherChannel between Core Switches
10. Configure IP addresses for all Core Switch interfaces
11. Configure IP addresses for all Distribution Switches
12. Configure Management IP addresses on Access Switches via VLAN 10
13. Configure Rapid PVST+ on Distribution & Access Switches
14. Enable PortFast & BPDU Guard on all active access ports
15. Configure OSPF for R1, Core Switches, & Distribution Switches
16. Enable DHCP
17. Enable SSH access 
18. Configure Extended ACLs 
19. Configure Port Security
20. Configure User Accounts
21. Save the Running-Config
</p>




<p><h3>1. Setup Hostnames</h3>

- In this step, we will set up the hostnames of each switch & router through their CLIs. In the picture below, you can see that I entered global configuration mode and entered “hostname AS-SW1” on the first access-layer switch in the IT department. </p>
  
<img src="https://i.imgur.com/N147OW0.png"/>
<p>
- Now repeat this process for each network device via the CLI.
- We will also configure a user account, secrets, & an inactivity timeout at the end of the project for security purposes. </p>

<p><h3>2. Configure EtherChannels in each Department</h3>

- In this step, we will be configuring VLANs & EtherChannels for the access layer & distribution layer switches
- Because we are using Cisco devices, we will be using PAgP (port aggregation protocol) for our EtherChannels. We are configuring these because this will increase the overall bandwidth between each configured device, as well as provide redundancy in case an interface stops functioning properly. 
- Let's start forming an EtherChannel between DS-SW1 and DS-SW2 in the IT department. In the CLI of DS-SW1, enter the command “show cdp neighbors” in privileged EXEC mode:</p>

<img src="https://i.imgur.com/XCJ8pTS.png"/>
<p>
- This allows us to see which interfaces are connected to other devices. Note that all of these only run Fast-Ethernet ports due to the Cisco model. In modern networks, all of these would be running at least on 1 Gigabit ports.
- From this command, we can see that DS-SW1 is connected to DS-SW2 via its F0/4 & F0/5 interfaces. 
- In global configuration mode, enter “interface range F0/4 - 5” to configure both interfaces at the same time.
Enter “channel-group 1 mode desirable”. This will put these interfaces in port channel 1 and make the interfaces actively try to form an EtherChannel with DS-SW2.</p>

<img src="https://i.imgur.com/AcmmQQq.png"/>
<p>
- Repeat these commands on DS-SW2’s interfaces connected to DS-SW1. Once complete, you can use the “show etherchannel summary” command to confirm that an EtherChannel has been formed.</p>

<img src="https://i.imgur.com/vBVhsQp.png"/>
<p>
- Notice the protocol is PAgP & next to the port channel SU appears, which means that the EtherChannel is currently in use. Additionally, the P next to the ports means that they are in the port channel. 
- Repeat this process for DS-SW3 & DS-SW4 in the HR department.</p>


<p><h3>3. Configure Trunk Ports</h3>

- In this step, we will configure all links between access switches & distribution switches as trunk links. This is so that we do not have to have a separate ethernet cable connection on each switch for each VLAN, but rather a trunk will allow all VLANs to travel via one connection (802.1Q tags will indicate the VLAN). The core switches will not have trunks as it is meant for pure speed, therefore we will have the distribution switches do all of the inter-VLAN routing.
- Note that we will also be setting the native VLAN to an unused VLAN as a security measure. This protects the network from VLAN-hopping attacks. Also, if the native VLAN was also used for management of the devices and there was a misconfiguration that allowed an unauthorized device to connect to the access-switch, this could allow the device to make management changes and execute malicious attacks. 
- As an overview, we will allow VLAN 10 (Management), 20 (Employees), & 30 (Interns) on all trunk ports for the IT department. For the HR department, VLAN 10, 30, & 40 (Employees) will be allowed. VLAN 90 will be our unused native VLAN, which will not be allowed on the trunk links. 
- Starting in the IT department, let's start on DS-SW1. Enter “show cdp neighbors” to show which interfaces on the switch is connected to DS-SW2 as well as the 3 access-layer switches.</p>

<img src="https://i.imgur.com/zpOby1n.png"/>
<p>
- From the picture you can see that the 3 access-layer switches in the IT department are connected to the f0/1, f0/2, & f0/3 interfaces of DS-SW1.
- Enter “interface range f0/1 - 3” in global configuration mode to configure all interfaces at once. Then enter the “switchport mode trunk” command to make them trunks (Note, you may run into a Command Rejected response if the interfaces were already running Dynamic Trunking Protocol (DTP). To fix this, change the interfaces to access ports, disable DTP via the “switchport nonegotiate” command, and then enter the “switchport mode trunk” command again. </p>

<img src="https://i.imgur.com/TZwsJvm.png"/>
<p>
- To set the native VLAN on the trunk links, enter “switchport trunk native vlan 90”
- To allow the rest of the VLANs, enter “switchport trunk allowed vlan 10,20,30”.
- We have now completed the trunk links from DS-SW1 to all 3 of the access-layer switches. To configure the port channel as trunks, enter “interface po1”, then repeat the commands above. Repeat all steps for DS-SW2, AS-SW1, AS-SW2, & AS-SW3 (access-layer switches will not have port channels). 
- The steps for the HR department are the same, except that there is VLAN 40 instead of VLAN 20. 
</p>

<p><h3>4. Configure VLANs via VTP</h3>

- In this step, we will set up VTP on one of the distribution switches for each department. We will do this because enabling VTP would allow the distribution switch to act as a server and the rest of the switches clients in the domain. Instead of manually configuring VLANs on each device, the distribution switch can update the rest of the switches on which VLANs are configured and update them automatically. This allows all switches to stay up to date with VLANs as well as prevent any manual misconfigurations.
- In the IT department, we will configure VTP on DS-SW1. In the command line, enter “show vtp status”. This will show us the current state of if VTP is currently in use or not.</p>

<img src="https://i.imgur.com/G3xcbnh.png"/>
<p>
- Notice how the VTP Operating mode is already set as Server. This is the default for switches. However, there is no VTP domain set, and the current VTP version is 1. 
- To configure a VTP domain name, enter global configuration mode, and enter the command “vtp domain Cisco”. This will create the Cisco VTP domain. Then enter “vtp version 2” to make the switch use version 2. We want version 2 because it can support extended VLANs beyond 1005. 
- After these configurations, DS-SW1 should start sending VTP advertisements to the other switches, in which they should automatically join the Cisco domain. You can verify this by going to AS-SW1 and entering “show vtp status”.</p>

<img src="https://i.imgur.com/88YgYKr.png"/>
<p>
- Notice how the VTP Domain Name is Cisco and is running version 2. However, the VTP Operating Mode is the default Server option, and we only want DS-SW1 to be able to add VLANs to the IT department network. To change the VTP Operating Mode to client, enter the command “vtp mode client” in global configuration mode. 
- Repeat this command for the rest of the switches (besides core) in the IT department.
- In the HR department, we will do the exact same steps, configuring DS-SW3 as the VTP Server and creating the VTP Domain Cisco.
- Now, we will create & name the VLANs on the VTP server. By doing this, all other switches that have joined the domain should automatically be configured with the VLANs as well. 
- In the DS-SW1 command line, enter “vlan 10”, then “name Management” in global configuration mode. Then, repeat this process for vlan 20 & 30. We can verify that the VLANs and their names were created with the “show vlan brief” command. </p>


<img src="https://i.imgur.com/7Sgpy38.png"/>
<p>
- We will repeat the above commands for the HR Department on DS-SW3, except we will have Vlan 40 instead of 20 for employees. 
</p>

<p><h3>5. Configure Access Ports</h3>

- In this step, we will manually configure the access ports on all access-layer switches to make sure they are operating in access mode. We will also be managing which access ports are assigned to each VLAN. 
- We will start in AS-SW1 in the IT department. Access ports are by default set to dynamic auto, meaning if they are connecting to another interface in dynamic desirable mode, the link will form a trunk. To stop this, we will disable DTP and manually enable access mode. In the command line, enter “show interfaces status” in privileged EXEC mode. This will allow us to see which interfaces are connected to the PCs.</p>

<img src="https://i.imgur.com/epIYa2h.png"/>
<p>
- We can see that f0/1 & f0/2 are connected to the workstations via the Status column, as the rest of the ports are not connected and f0/3 & f0/4 are our trunk ports. 
- In the command line, enter “interface range f0/1 - 2”, then “switchport mode access” to enable access mode. This will automatically disable DTP. 
- The workstations connected to AS-SW1 & AS-SW2 will both be used by IT employees, so therefore enter the command “switchport access vlan 20”.
- Repeat these steps for AS-SW2 & AS-SW3, except that interns will be using the workstations connected to the AS-SW3. 
- Now repeat this process for the HR department. AS-SW6 will be using VLAN 30 for interns. 
</p>

<p><h3>6. Disable Unused Ports</h3>

- In this step, we will disable all unused ports on access-layer & distribution layer switches. Let’s start on DS-SW1 again with the command “show interfaces status” in privileged EXEC mode. </p>

<img src="https://i.imgur.com/zooqXUA.png"/>
<p>
- We can see from this command that interfaces from f0/6 to g0/2 are not connected to anything and therefore should be administratively disabled for security purposes. 
- To disable these interfaces, enter the command “interface range f0/6-24, g0/1-2” in global configuration mode, then enter “shutdown” to disable the interfaces.</p>

<img src="https://i.imgur.com/IZYuxBu.png"/>
<p>
- From the picture above, you can see that they are now labeled as disabled in the Status column. Repeat this process for the rest of the switches including the HR department.
</p>

<p><h3>7. Configure R1’s IP addresses</h3>

- We are now moving on to Layer 3 configuration of our enterprise network. We will start off with enabling the interfaces connected to each department’s core switches. On R1’s CLI, enter “show ip interface brief” and then “show cdp neighbors”.</p>

<img src="https://i.imgur.com/ps4y9kh.png"/>
<p>
- This is different from Cisco switches, as Cisco routers have all interfaces set to administratively down by default, while switch interfaces will be in the up status when connected to another interface.
- To start with the interface connected to the IT department, enter the command “interface g0/0” and then “ip address 172.16.1.1 255.255.255.0”, then “no shutdown” to enable the interface.</p>

<img src="https://i.imgur.com/4w0xgpY.png"/>
<p>
- For the interface connected to the HR department, g0/1, do the same commands but for the ip address of 172.16.2.1 /24. 
- We will also create a loopback interface on R1, which is beneficial in case an interface shuts down unexpectedly and the IT or HR department need another way to reach the router. This is a virtual interface, so it does not need to be tied to one of R1’s physical interfaces. 
- In the command line, enter “interface l0” in global configuration mode. Notice that the virtual interface was enabled by default. Then, enter “ip address 1.1.1.1 255.255.255.255” to give it an ip address. Let us confirm that each interface is assigned correctly with “do show ip interface brief”. </p>

<img src="https://i.imgur.com/R69XwUF.png"/>

<p><h3>8. Enable IP Routing on all Distribution & Core switches</h3>

- In this step, we will enable the distribution & core switches of each department to be able to use IP routing, since they are multilayer switches. This is useful so that the distribution switch can do inter-VLAN routing on its own instead of having to rely on R1. 
- On each distribution & core switch, enter “ip routing” in global configuration mode.
</p>

<p><h3>9. Configure PAgP EtherChannel between Core Switches</h3>

- This step will be similar to when we created an EtherChannel between the distribution switches, but at layer 3. 
- On CS-SW1, enter “show cdp neighbors” to see which interfaces are connected to CS-SW2. </p>

<img src="https://i.imgur.com/o7FZlVt.png"/>
<p>
- F0/5 & 6 are connected to CS-SW2. Enter “interface range f0/5-6”, then enter “no switchport” to make them routed ports. Finally, enter “channel-group 1 mode desirable”. We then must configure an ip address on the port channel interface for layer 3 EtherChannels. To do so, enter “interface po1”, then “ip address 172.16.3.1 255.255.255.0”.
- On CS-SW2, repeat the same steps except with an IP address of 172.16.3.2 /24. 
- Using the command “show etherchannel summary” will allow us to confirm that the PAgP EtherChannel was successful. </p>

<img src="https://i.imgur.com/tY2V3Ec.png"/>
<p>
- RU next to the port channel means that it is through layer 3 and it is successfully in use. 
- You can also ping CS-SW2’s IP address from CS-SW1 CLI to confirm that they are able to reach each other. 
</p>

<p><h3>10. Configure IP addresses for all Core Switch interfaces</h3>

- In this step, we will manually configure IP address for each active interface, then disable any interfaces that are unused. 
- Enter the command “show cdp neighbors” again on CS-SW1. We need to configure F0/1-4, as well as G0/1. 
- Enter the IP address for each interface as shown. </p>

<img src="https://i.imgur.com/1g9yc1h.png"/>
<p>
- For all other interfaces, enter the “shutdown” command to disable them.
- Do the same configuration for CS-SW2 with the IP addresses shown.</p>

<img src="https://i.imgur.com/vZIhthR.png"/>

<p><h3>11. Configure IP addresses for all Distribution Switches</h3>

- For DS-SW1, configure the IP addresses shown below. </p>

<img src="https://i.imgur.com/145DWKr.png"/>
<p>
- For DS-SW2, configure the IP addresses as shown (project 21). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- For DS-SW3, configure the IP address as shown (project 22)</p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- For DS-SW4, configure the IP addresses as shown (project 23)</p>

<img src="https://i.imgur.com/vzNHH42.png"/>

<p><h3>12. Configure Management IP addresses on Access Switches via VLAN 10</h3>

- The purpose of configuring management IP addresses on access switches is so that if we are not on-site and need to change settings/configuration, we have a way to enter the switch remotely via SSH. We also want to configure a default gateway so that SSH can work properly. 
- AS-SW1-3 will all be in the 172.16.12.0 /24 network, with the default gateway being 172.16.12.1 /24. Starting in AS-SW1, enter the command “ip default gateway 172.16.12.1” in global configuration mode to set the gateway. Then, enter “interface vlan 10” to enter the management VLAN. Finally, enter “ip address 172.16.12.2 255.255.255.0” for the switch virtual interface (SVI) ip address. (project 24). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- Do the same for the other 2 access switches, with .3 & .4 ip addresses instead.
- For AS-SW4-6, the network will be 172.16.13.0 /24, with a default gateway of 172.16.13.1
- Repeat the same process as the IT department but for the .13 network.
</p>

<p><h3>13. Configure Rapid PVST+ on Distribution & Access Switches</h3>

- The purpose for configuring rapid per-vlan spanning-tree protocol is to prevent broadcast storms & loops at layer 2. By default, the Cisco switches used in this enterprise network will have PVST+ running by default, but not Rapid PVST+. We want Rapid PVST+ so that it doesn’t take as long for each switch interface to determine if they can forward traffic or be in a blocking state. 
- Starting on DS-SW1, enter “spanning-tree mode rapid-pvst” in global configuration mode. You can check if the switch is using Rapid-PVST+ by entering the “show spanning-tree” command. On the top, it should say protocol rstp per VLAN. (project 25). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- Repeat these commands for each distribution & access switch. 
</p>

<p><h3>14. Enable PortFast & BPDU Guard on all active access ports</h3>

- Enabling PortFast on access ports connected to end hosts allows the interfaces to transition directly to a forwarding state, as it assumes that no switches are connected to it and therefore will not cause any loops. Enabling BPDU Guard is beneficial in case someone accidentally plugs in another switch to one of these access ports. BPDUs are sent from switches for spanning-tree, so if BPDU Guard detects this on an access port, it will shutdown the interface. 
- To do so, all we have to do is enter the commands “spanning-tree portfast” & “spanning-tree bpduguard enable” in global configuration interface mode for the interfaces connected to end hosts on each access switch. 
</p>

<p><h3>15. Configure OSPF for R1, Core Switches, & Distribution Switches</h3>

- OSPF is a dynamic routing protocol that allows all devices configured with OSPF to draw a network layout of the enterprise network. This process involves layer 3 devices sharing all of their known routes and then choosing the best possible route to destinations (sometimes routers will use more than one path if they are the same metric and load balance each path accordingly).
- Starting on R1, we will enable OSPF on each interface connecting to the LAN. We will be using process ID 1 & area 0 for this enterprise network. In global configuration mode, enter “router ospf 1” to enter the ospf configuration mode. Then, enter “network 172.16.1.0 0.0.0.255 area 0” to enable ospf on R1’s g0/0 interface. Note that OSPF uses wildcard masks, not subnet masks. We can do the same command for the g0/1 interface but with its network and the loopback (l0) interface. For the loopback interface, we will use the “passive-interface l0” command, as we do not need the loopback interface to send OSPF hello messages out of its interface constantly. (project 26)</p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- We will then repeat this process on all Layer 3 interfaces for the rest of the network. On DS-SW1 for example, you can enter the “show ip route” command in privilege EXEC mode to see all of the routes added via OSPF (project 27). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- However, if you look at the output you can see that the gateway of last resort is not set, meaning no default route has been created. We will make one on R1 to go out of its g0/2 interface to the internet. 
- On R1’s CLI, use the command “ip route 0.0.0.0 0.0.0.0 203.11.0.2” to create a default route to the internet. Then in ospf global configuration mode, enter “default-information originate” which will share the default route to the other ospf enabled devices. You should then see that a gateway of last resort has been set. (project 28). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- R1 advertising its default route configures the router as an autonomous system boundary router (ASBR), which is the router between the backbone area (0) and the external network (the internet). 
</p>

<p><h3>16. Enable DHCP</h3>

- The purpose of configuring DHCP is so that the end host in our enterprise network can receive IP addresses automatically, as well as other information such as default gateway.
- We will enable DHCP on R1, in which it will act as a DHCP server for the VLANs we configured earlier. 
- We will start with the management VLAN 10 for the IT department. In this VLAN, the subnet will be 172.16.14.0/24 with a default gateway as 172.16.14.1. The DNS server will use an external DNS server, 8.8.8.8 (Google’s DNS server). 
- Additionally, we will exclude the first 10 addresses in each DHCP pool, in case we need those addresses later for additional network devices. 
- In the R1 CLI in global configuration mode, enter “ip dhcp excluded-address 172.16.14.1 172.16.14.10” to exclude the first 10 address in the VLAN. 
- Then, enter “ip dhcp pool IT-management” to create the dhcp pool, and then “network 172.16.14.0 255.255.255.0” to specify the network. 
- Use the command “default-router 172.16.14.1” to configure the default-gateway. 
- Use the command “dns-server 8.8.8.8” to configure the DNS server (project 29) </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- After this, we will configure the rest of the VLANs the same way but with their respective addresses and gateways. For IT VLAN 20, the network will be 172.16.15.0/24 with a default gateway of 172.16.15.1. For IT VLAN 30, the network will be 172.16.16.0/24 with a default gateway of 172.16.16.1. For the HR VLAN 10, the network will be 172.16.17.0/24 with a default gateway of 172.16.17.1. For the HR VLAN 30, the network will be 172.16.18.0/24, with a default gateway of 172.16.18.1. For the HR VLAN 40, the network will be 172.16.19.0/24, with a default gateway of 172.16.19.1. 
</p>

<p><h3>17. Enable SSH access</h3>

- SSH will allow us to remotely access a network device for management over an encrypted connection. In our enterprise network, only the IT employees should be able to do this, which is subnet 172.16.15.0/24. Therefore, we will enable SSH on all switches & routers as well as create a simple ACL that will permit that subnet to SSH into these devices, but deny all other connections. 
- Starting in R1, enter “crypto key generate rsa” in global configuration mode, and enter 4096 for the largest key size, meaning the most secure. (project 30) </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- Next, enter “access-list 1 permit 172.16.15.0 0.0.0.255”. This will permit the IT employees to SSH into the router, and implicitly deny any other traffic. 
- We will now enable vty lines, which are logical interfaces that allow a device such as R1 to be connected using SSH. 
- In the CLI, enter “line vty 0 15”, then “access-class 1 in”. The last command applies the access control list we previously created to our vty lines. 
- We only want SSH connections, not Telnet, as the latter is not secure since it does not encrypt traffic. To do this, enter “transport input ssh”. Finally, we will force the user using SSH to use a local account (the local account will be set up in the last step). To do this, enter “login local” and then “logging synchronous”. (project 31). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- Now, repeat these commands for the rest of the network devices. 
</p>

<p><h3>18. Configure Extended ACLs</h3>

- In this step, we will configure a simple extended ACL so that workstations in the Interns subnet in each department are unable to send traffic to each other (except ICMP traffic). Currently, the DS-SW1 is the default gateway for the interns VLAN for the IT department, while DS-SW3 is the default gateway for the interns VLAN for the HR department. Therefore, we will apply the extended ACLs there. 
- Starting on DS-SW1, we will create the access control list by entering “ip access-list extended Interns” in global configuration mode.
- We will first start with the most specific part, allowing ICMP messages between the Interns subnet. Enter “permit icmp 172.16.16.0 0.0.0.255 172.16.18.0 0.0.0.255”. 
- Next, we will deny the rest of the traffic between the subnets using the command “deny ip 172.16.16.0 0.0.0.255 172.16.18.0 0.0.0.255”. 
- Finally, we will permit the rest of the traffic to flow through as normal with “permit ip any any”. (project 32) </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- We then apply this to VLAN 30 by entering “interface vlan 30”, then “ip access-group Interns in”. (Note that the picture above is from R1. This is a mistake and should be made for DS-SW1, as this is the default gateway for the subnet). We can then reverse the ip address in the extended access list for DS-SW3. 
</p>

<p><h3>19. Configure Port Security</h3>

- The purpose of configuring port security is to prevent malicious users from accessing our enterprise network by plugging in their device to an access port. Port security will allow us to limit the number of MAC addresses per port to 1, as well as configure a violation mode that will automatically shutdown the port and send us a message (via syslog or SNMP). We will configure port security for the interfaces our workstations are connected to per access-switch, f0/1 & f0/2. 
Starting with AS-SW1, enter “interface range f0/1-2” in global configuration mode.
Then, enter “switchport port-security” to enable port security. The default violation mode is shutdown, in which a different MAC address than what is already configured as safe connects to the interface will cause the port to enter the down state and send an SNMP notification. Therefore, we will keep it in its default violation mode. (project 33).</p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
If we wanted to specify what MAC address was allowed on a port, we can use the command “switchport port-security mac-address xxxx.xxxx.xxxx”.
</p>

<p><h3>20. Configure User Accounts</h3>

- Switches & routers need to be secure from people accessing the console port, privileged EXEC mode, & global configuration mode. We can do this by creating a user account & secret (password) and setting the console line to require logging in with this user account. 
- Starting with R1, enter “enable secret Cisco” in global configuration mode. Note that “enable password” will not hash the password you set, and therefore this can be seen in plaintext in the startup-config file. 
- This command we just entered sets a password for those who are trying to enter privilege EXEC mode. 
- Next, in global configuration mode enter “user cisco secret ccna”. This creates the user account & password that will be needed to SSH into the network devices as well as use the console port. 
- To configure the console line, enter “line console 0” since there is only one console line for R1. Entering “login local” will require the user to login with the user account we created. Additionally, we can enter “exec-timeout 15” to automatically log the user out of the console port when there is 15 minutes of inactivity, increasing the enterprise network’s security posture. (project 34). </p>

<img src="https://i.imgur.com/vzNHH42.png"/>
<p>
- We will then apply these commands to all switches in the enterprise network.
</p>

<p><h3>21. Save the Running-Config</h3>

- Finally, we must ensure that all of our configurations are saved to the startup-config file. If the network devices reboot, none of our configurations will load as it has only been stored on the running-config file so far.
- On each network device’s CLI, all you have to enter is “write memory”. You can verify that it was saved via the “show startup-config” command. 
</p>

<p><h3>Improvements for this Project/Configuration</h3>

- This project is in-depth and lengthy, therefore some configurations are left out that could improve this LAN architecture. 
For additional security, we could configure dynamic ARP inspection, which relies on DHCP snooping which we also did not configure. DHCP snooping is layer 2 security in which switches keep track of devices that are sending out DHCP offers and disable devices that are untrusted in the network. Due to DHCP snooping creating a table of valid MAC to IP addresses, dynamic ARP inspection can use this to detect if ARP is poising being attempted. 
Another improvement is to add additional DNS servers to our network, whether it is an internal or external server. This configuration could provide redundancy in case one of the DNS servers were to experience an outage. 
Finally, creating documentation on the network logical topology would be extremely useful for troubleshooting, further configuration, & many other uses for the networking team. 
</p>
