# HSRPv2 & RSTP Synchronization

# Overview
In this Project I created a network on GNS3, and it features FHRP & STP synchronization design, HSRPv2 as FHRP and RSTP.
HSRP is short for (Hot Standby Routing Protocol) and it is a **First Hop Redundancy Protocol** which is used to set multiple gateways for a subnet, one will act as a master or active and the other is in standby mode; waiting for the active Router (gateway) to fail, and if it fails it will become active.
STP (Spanning-Tree Protocol) is the protocol used to prevent loops and make linking rules for the switches on the network, it is essential for a network that has many switches connected, but still it need attention to configuration detail.
## Topology
![Topology](/topology.png)
As you can see this is the topology of the network.
## Features:
* 4 VLANs, 10, 20, 30, 40.
* DHCP and DNS servers on vlan 40, configured staticaly.
* Relay-agent configuration on all distrubution MultiLayer Switches (MLS1, MLS2).
* Spanning-Tree Basic configuration, MLS3 is root for VLAN 10 and 20, and MLS4 is root for VLAN 30 and 40, to make sure that traffic go through the must direct path to the destination and don't get flipped and looped with Spanning-tree.
* HSRP version 2 with preemption on MLS3 and MLS4, MLS3 will be the active for VLAN 10 and 20 and standby for others, and MLS4 will be standby for VLAN 30, and 40 and standby for others, this will make sure that even if the gateway fails the traffic will go through the other gateway until their gateway comes back.
* OSPF with single area configuration on MLS3, MLS4, R1 and R2, this you can call the core layer; as in the topology it is the head of the network and it is connected to the cloud which mean Internet connection or WAN connection (to connect to another site), so OSPF is good Routing Protocol for this.
    * Neighbor Adjacencing on R1 to MLS3 + MLS4.
    * and R2 to MLS4 + MLS3.
****
**in the planning.txt file at (/Planning/full-plan.txt) you can see all the network planning from scratch**
--> [full-plan](/planning/full-plan.txt)

****
or run:
```git bash
git clone https://github.com/HootaNet/FHRP-STP-Sync
cd planning
run planning.txt
```

* You can see all the configuration in text files at (/Planning/Devices-configuraiton), they are used to get all the configuration commands ready before applying them on the IOS,this is a useful automation technique, and it is flexible to make changes.
* Also you can copy or view the configuration files for each device within the topology at the (Devices-config-files) directory.
### Planning Configuration Text Files:
* [MLS1 config plan](/planning/Devices-Configurations/MLS1.txt)
* [MLS2 config plan](/planning/Devices-Configurations/MLS2.txt)
* [MLS3 config plan](/planning/Devices-Configurations/MLS3.txt)
* [MLS4 config plan](/planning/Devices-Configurations/MLS4.txt)
* [R1 config plan](/Planning/Devices-Configurations/R1.txt)
* [R2 config plan](/Planning/Devices-Configurations/R2.txt)
* [DHCP-server config plan](/Planning/Devices-Configurations/DHCP-Server.txt)
* [DNS-server config plan](/Planning/Devices-Configurations/DNS-Server.txt)
****
### Devices Configuration Text Files:
* [MLS1 config file](/Devices-Config-Files/MLS1-config-file.txt)
* [MLS2 config file](/Devices-Config-Files/MLS2-config-file.txt)
* [MLS3 config file](/Devices-Config-Files/MLS3-config-file.txt)
* [MLS4 config file](/Devices-Config-Files/MLS4-config-file.txt)
* [R1 config file](/Devices-Config-Files/R1-config-file.txt)
* [R2 config file](/Devices-Config-Files/R2-config-file.txt)
* [DHCP-Server config file](/Devices-Config-Files/DHCP-Server-config-file.txt)
* [DNS-Server config file](/Devices-Config-Files/DNS-Server-config-file.txt)
****
## Connectivity Map
![connectivity-map-screenshot](/connectivity-map/screenshot.png)
here is a screenshot to the spredsheet that contains full connectivity map for all devices, you can view what device is connected to which device with which interface.
***
# Verification
In this section I will show verification of the most important configuration, and screenshots of show commandss output.
***
![screenshot0](/Screenshots/0.png)
![screenshot1](/Screenshots/1.png)
![screenshot2](/Screenshots/2.png)

Here you can see that PC2 on VLAN 10, PC3 on VLAN 20, PC5 on VLAN 30 all are able to get IP addresses from the DHCP Server on VLAN 40, and all can ping their gateway.
***
![screenshot3](/Screenshots/3.png)
The above screenshot shows that PC1 can ping PC3 on VLAN 20 which has the IP 10.0.20.11, and PC5 on VLAN 30 which has the IP 10.0.30.11, and the DHCP-Server with IP address 10.0.40.10, and DNS-Server with IP address 10.0.40.20.
***
![screenshot4](/Screenshots/4.png)
![screenshot5](/Screenshots/5.png)
From these two screenshots you can see that MLS3 is running HRSP version 2 and it is active for VLAN 10 and VLAN 20, and it is standby for VLAN 30 and 40 and you can see some IP addresses configuration. Also MLS4 is running HSRP version 2 and it is active for VLAN 30 and VLAN 40, and it's standby for VLAN 10 and VLAN 20, and you can see some IP addresses configuration.
***
![screenshot6](/Screenshots/6.png)
![screenshot7](/Screenshots/7.png)
The two above screenshots shows some Spanning-Tree information, MLS3 is root for VLAN 10 and 20, and MLS4 is root for VLAN 30 and 40, not showing but MLS1 has better priority (lower) for VLAN 10 and 20 than MLS2, and MLS2 has lower priority for VLAN 30 and 40 from MLS1, so for example if MLS3 fails the traffic path from VLAN 10 and 20 will be through MLS1-MLS4 and R1 or R2
***
![screenshot8](/Screenshots/8.png)
![screenshot9](/Screenshots/9.png)
And here some OSPF information such as shared networks on the routing table, neighbor adjacency, and IP addresses. OSPF is also enabled on MLS3 and MLS4, the connection between MultiLayer Switches to the Routers is Point-to-Point connection, so a MultiLayer Switch is connected with two Routers, so it's neighbor with the two routers, this is also a redundancy technique.
***
# Finishing
The purpose of HSRP and STP synchronization is that traffic goes through the most direct path, because by default Spanning-Tree is a double-edged protocol!, it is made to prevent loops, but however, it can create it's own loops due two wrong configuration or mismatches between switches. Spanning-Tree also blocks some links that might be important by default, by using PVST+ configuration it will allow load-balancing for multiple vlans.

I want to point out that I had A lot of problems while I'm experiencing this project, the GNS server died, so I had to remove it from VirtualBox and install it again, also I updated the configuration planning text files a lot because some commands orders wasn't right, and some commands were missing so I added them.

But yeah!, I'm proud of this project and it was fun despite the issues I had and struggles.

And Thank you for viewing this Project.
------------------ Best wisshes Mahmoud Omar (HootaNet)