
# C9800 Next Gen Lab Ramblings

Core to this lab is running the C9800-CL in a KVM enviroment on an Intel NUC platform.   This combination was selected due to the compact nature of the NUC and the low power and noise of the platform as-well.  KVM on Ubunut was selected due to the dirver support for the NUC platform and the ability to run other services on the host (dhcpd, dns etc.) VMWare requires licenses, VCenter, and special driver builds for the Intel NUC.

## Lab Organization

ASA - Outside / NAT services Outside interface does DHCP , intent is to connect to Cisco / Partner network to connect the LAB to the internet.  For Cisco Networks please request IOT access for the MAC address of the Outside interface G/1 from the eSTORE application.

C9300-24 Main Router/switch for the network.  

Two NUC platforms running 32GB RAM , 1TB HD and QuadCore 8th Generatioin i7s are the base build.  

LABNUC01 is running isc-dhcp-server, ntp, bind9 and KVM with the following Virtuial Machienes 

- ISE
- POD00 - Master POD
- POD01 - POD04  
LABNUC02 is running isc-dhcp-server (failover) ntp KVM
- POD05 - POD08

### VLAN Structure

VLAN 10 - Management VLAN , This is where all the Students will end up when they connect to the Lab SSID @9800-LAB

- VLAN 1 - Host Managment ports for NUC hosts.
- VLAN 1x1 - Pod AP where x = pod number
- VLAN 1x0 - Pod Clients where x = pod number

### PODs

- VM POD00 - Master Pod for all the students to connect to.
- VM ISE - ISE Node - 8GB RAM - 200GB HD Space
- VM POD01 - POD08  Student Pods - 4GB RAM 8GB 

***C9800-CLs configured with 4gb ram 8gb of Disk 4gb is lower then minimum requiremtnt but i have not experienced any issues so far.***

#### Pod Networking



### Infrastructure

#### ASA

ASA is configured with three interfaces, Inside , Outside and MGMT.

#### C9300-24

The switch is configured with one managment VLAN , VLAN 10 - 10.10.10.XXX subnet , and then two vlans per pod, 10.10.1x0 and 10.101.1x1 little x being pod number 1-8  clients on pod .1x0 and APs on .1x1

## Key Files

- **./IOS-XE-Confgs** folder contains the lab pod configurations and C9300 Configurations and ASA , Yes I know its not IOS-EX but.. 
- **./Procedures** folder contains guides on startup and shutdown of the pods.
- **./Initialization** folder contains procedures for initial set up of the lab and initialization of the pods.
  - **./Initialization/01-NUC-LAB-Initalization.md** file explaining how to set up the Intel NUC with Unbuntu, KVM and network Services.
  - **./Initialization/WLC_POD_Initialization_commands.ios** file with the base commands to bootstrap 9800-CL with      out going through the DAY0 Config Wizzard.  
- **../Sample Files** folder contains example files.

## Lab Tips

Just a few suggestions, not necessarly required to get things working but they will help.

First find a good text editor, Like VIM, Atom, Sublime, or Visual Code.  Although primarly for codeing they can help with formating xml, dhcpd.conf, and Cisco Config files (.ios) in my folders.  

Second , download from Cisco.com the Cisco CLI Analyzer its a good tool for launch ssh CLI Sessions, expecially if your managing the LAB and need to quickly access a students POD.

If you need to move files around, I have found that **scp** (secure copy) is the easiest way. Cisco IOS, Apple MACOS, and Linux nativly support it. For Microsoft Windows users, there are tools available to do **scp** from Microsoft Windows.