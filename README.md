
# C9800 Next Gen. Lab Ramblings

Core to this lab is running the C9800-CL in a KVM environment on an Intel NUC platform.   This combination was selected due to the compact nature of the NUC and the low power and noise of the platform as-well.  KVM on Ubunut was selected due to the driver support for the NUC platform and the ability to run other services on the host (dhcpd, dns etc.) VMWare requires licenses, VCenter, and special driver builds for the Intel NUC. 

## Lab Organization

ASA - Outside / NAT services Outside interface does DHCP , intent is to connect to Cisco / Partner network to connect the LAB to the internet.  For Cisco Networks please request IOT access for the MAC address of the Outside interface G/1 from the eSTORE application.

![High Level Architecture](https://github.com/naboyd/C9800-LAB/blob/master/Cat9kLABDrawings/Architecture.png)

C9300-24 Main Router/switch for the network.  

Two NUC platforms running 32GB RAM , 1TB HD and QuadCore 8th Generation i7s are the base build.  

LABNUC01 is running isc-dhcp-server, ntp, bind9 and KVM with the following Virtual Machines

- ISE
- POD00 - Master POD
- POD01 - POD04  
LABNUC02 is running isc-dhcp-server (failover) ntp KVM
- POD05 - POD08

### VLAN Structure

VLAN 10 - Management VLAN , This is where all the Students will end up when they connect to the Lab SSID @9800-LAB

- VLAN 1 - Host Management ports for NUC hosts.
- VLAN 1x1 - Pod AP where x = pod number
- VLAN 1x0 - Pod Clients where x = pod number

### PODs

- VM POD00 - Master Pod for all the students to connect to.
- VM ISE - ISE Node - 8GB RAM - 200GB HD Space
- VM POD01 - POD08  Student Pods - 4GB RAM 8GB 

***C9800-CLs configured with 4gb ram 8gb of Disk 4gb is lower then minimum requirement but i have not experienced any issues so far.***

#### Pod Networking

The VM Guests connect to the KVM network interface br0 which is bridged to the linux interface br0 (just to add confusion).  This connectivity allows the Cisco C9800-CL PODS to build a 802.1q trunk off their GigibitEthernet 1 interface. Cisco Identity Services Engine (ISE) just uses the Native VLAN on its GigibitEtherent 1 interface to talk to the switch.

![Pod Networking Architecture](https://github.com/naboyd/C9800-LAB/blob/master/Cat9kLABDrawings/Host%20Architecture.png)


### Infrastructure

#### ASA

ASA is configured with three interfaces, Inside , Outside and MGMT.

#### C9300-24

The switch is configured with one management VLAN , VLAN 10 - 10.10.10.XXX subnet , and then two VLANS per POD, 10.10.1x0 and 10.101.1x1 little x being pod number 1-8  clients on pod .1x0 and APs on .1x1

![Switch Port Layout](https://github.com/naboyd/C9800-LAB/blob/master/Cat9kLABDrawings/Switch%20Port%20Layout.png)

## Key Files

- **[./IOS-XE-Confgs](https://github.com/naboyd/C9800-LAB/tree/master/IOS-XE-Configs)** folder contains the lab pod configurations and C9300 Configurations and ASA , Yes I know its not IOS-EX but.. 
- **[./Procedures](https://github.com/naboyd/C9800-LAB/tree/master/Procedures)** folder contains guides on startup and shutdown of the pods.
- **[./Initialization](https://github.com/naboyd/C9800-LAB/tree/master/Initialization)** folder contains procedures for initial set up of the lab and initialization of the pods.
  - **[./Initialization/01-NUC-LAB-Initialization.md](https://github.com/naboyd/C9800-LAB/blob/master/Initialization/01-NUC-LAB-Initalization.md)** is a file explaining how to set up the Intel NUC with Unbuntu, KVM and network Services.
  - **[./Initialization/WLC_POD_Initialization_commands.ios](https://github.com/naboyd/C9800-LAB/blob/master/Initialization/WLC_POD_Initialization_commands.ios)** file with the base commands to bootstrap 9800-CL with      out going through the DAY0 Config Wizzard.  
- **[./Sample Files](https://github.com/naboyd/C9800-LAB/tree/master/Sample%20Files)** folder contains example files.
- **[./Host_Configurations](https://github.com/naboyd/Host_Configurations)** contains the relevant host configuration files.

## Lab Tips

Just a few suggestions, not necessarily required to get things working but they will help.

1. Find a good text editor, Like VIM, Atom, Sublime, or Visual Code.  Although primarily for writing code they can help with formatting xml, dhcpd.conf, and Cisco Config files.  

2. Download from Cisco.com the Cisco CLI Analyzer its a good tool for launch ssh CLI Sessions, especially if your managing the LAB and need to quickly access a students POD.

3. If  you need to move files around, I have found that **scp** (secure copy) is the easiest way. Cisco IOS, Apple MACOS, and Linux nativly support it. For Microsoft Windows users, there are tools available to do **scp** from Microsoft Windows.

## Acknowledgements

Kudos to Matt Cone and his excellent [Markdown Guide](https://www.markdownguide.org/getting-started) this was my first crack at Markdown Documentation.  Second goes to David Anson, and his VSCODE markdownlint extension.  It keeps you on track with the rules of Markdown.

And of course , Catalyst 9800-CL is a product and brand of Cisco, Intel NUC, and Ubuntu Linux.

## Disclaimer

This is provided with no expressed or implied warranty.  This is just a guideline for building a lab / workshop environment.  This is by no means an endorsement of running the contained systems in a production environment as outlined here.  Cisco has production grade deployment guides on (Cisco.com)[https://www.cisco.com] that should be followed to ensure a supportable production ready deployment. 