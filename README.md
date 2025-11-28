# DHCP_and_Etherchannel
This lab demonstrates the DHCP relay agent concept and port aggregation using LACP.

# Overview

This repository contains a Cisco Packet Tracer lab demonstrating how to build a small enterprise network using:
- A  router works as DHCP server
- Multilayer Switch for SVI based Inter-vlan routong
- DHCP relay agent for connected to DHCP server on the router
- Static routing
- Port aggregation using LACP
- Trunking for crry multiple vlan traffic

# Network Topology

![Diagram](Diagram/Network_Topology.png)

# Steps for the Key configurations 

- configure a DHCP pools for each vlan on the router
  
  ```
  ip dhcp exclude-address 192.168.10.1
  ip dhcp pool vlan10
    network 192.168.10.0 255.255.255.0
    default-router 192.168.10.1
    exit
  
  ip dhcp exclude-address 192.168.20.1
  ip dhcp pool vlan20
    network 192.168.20.0 255.255.255.0
    default-router 192.168.20.1
    exit

  ip dhcp exclude-address 192.168.30.1
  ip dhcp pool vlan30
    network 192.168.30.0 255.255.255.0
    default-router 192.168.30.1
    exit

  ip dhcp exclude-address 192.168.40.1
  ip dhcp pool vlan40
    network 192.168.40.0 255.255.255.0
    default-router 192.168.40.1
    exit
  ```

  - Assigning ip address for the link connected to router from MLS
 
  ```
  [ In router ]
  interface gig0/0/0
    ip address 172.16.0.1 255.255.255.252
    no shutdown
  ```
  ```
  [ In MLS ]
  interface gig0/2
    no switchport
    ip address 172.16.0.2 255.255.255.252
    no shutdown
  ```

  - Set static routes on router for each vlan's SVI
 
  ```
  ip route 192.168.10.0 255.255.255.0 172.16.0.2
  ip route 192.168.20.0 255.255.255.0 172.16.0.2
  ip route 192.168.30.0 255.255.255.0 172.16.0.2
  ip route 192.168.40.0 255.255.255.0 172.16.0.2
  ```

  - In MLS creating SVIs and assigning the ip helper address for it

  ```
  vlan 10
  name vlan10 [can give a name for vlans but i didnt]
  interface vlan 10
    ip address 192.168.10.1 255.255.255.0
    ip helper-address 172.16.0.1
    exit
  ```
- Like this create the SVI for each vlans, then give the `ip routing` command in configuration mode.

- Then set the route for forward traffic to the router

  ```
  ip route 172.16.0.0 255.255.255.252 172.16.0.1
  ```

- then create the port channel

```
[ In MLS ]
interface range f0/1-3
  channel-group 1 mode active
  exit

interface port-channel 1
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  exit
```
```
[ In S1 switch ]
interface range f0/1-3
  channel-group 1 mode active
  exit

interface port-channel 1
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  exit
```
```
[ In MLS ]
interface range f0/4-6
  channel-group 2 mode active
  exit

interface port-channel 2
  switchport trunk encapsulation dot1q
  switchport mode trunk
  switchport trunk allowed vlan 30,40
  exit
```
```
[ In S2 switch ]
interface range f0/1-3
  channel-group 2 mode active [ Not problem with NO 1, I use 2 for not to confused ]
  exit

interface port-channel 2
  switchport mode trunk
  switchport trunk allowed vlan 30,40
  exit
```

- Then creating vlans on switches and assigning ports to them

```
[ In S1 switch ]
vlan 10
interface range fa0/4, fa0/6
  switchport mode access
  switchport access vlan 10
  exit

vlan 20
interface range fa0/5, fa0/7
  switchport mode access
  switchport access vlan 20
  exit
```
```
[ In S2 switch ]
vlan 30
interface range fa0/4, fa0/6
  switchport mode access
  switchport access vlan 30
  exit

vlan 40
interface range fa0/5, fa0/7
  switchport mode access
  switchport access vlan 40
  exit
```

- In here use `show ip interface brief` command to see the vlan status. If vlan are down go to the vlan interface and give `no shutdown` command.

- In each pc enable DHCP to get ip from the DHCP server.

## Note on Basic Security Configurations
This lab intentionally does not include the following initial security configurations:
-	console / VTY passwords
-	MOTD banners
-	SSH setup
-	privilege mode protection
-	password encryptions
- creating management vlan


## What this lab demonstrate
- How to use router as DHCP server
- How to set ip-helper address for access the DHCP server in a diffrent network
- How port aggregation done in a network to enable redunduncy and increase bandwidth
- How SVI use to enable inter vlan routing

## How to Use This Repository

-	Clone the repo
-	Open the .pkt file in Cisco Packet Tracer
-	Review configurations
-	Test connectivity between all VLANs
-	Modify, break, and rebuild the design to reinforce learning

    
  
