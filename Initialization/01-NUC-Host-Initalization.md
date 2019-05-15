
# Lab Host Initialization

These are instructions on how to initialize the Intel NUC host with Ubuntu 18.04 and KVM.  These are meant to be a guideline, there is no expressed or implied warranty that something might break. Your results may vary, if you run into issues setting up based on the below instructions, you will only need a couple of tools beyond this to assist. First is creative Googling, or DuckDuckGoing, filtering the results and a little time and application.  The modules and procedures below were selected for simplicity sake. This is by no means a cook book for a production network or network in which your career hangs in the balance with.  

***This is for a LAB environment only.***

## Platform Install

### Install OS- Ubuntu 18.04

Download from ubuntu.com and burn on an ISO

[Ubuntu 18.04 Desktop](https://www.ubuntu.com/download/desktop/thank-you?country=US&version=18.04.1&architecture=amd64 )

Burn the ISO to a USB Stick. I have had great success with Etcher

[Balena Etcher](https://www.balena.io/etcher/)

Boot platform from USB Stick and follow install instructions .
The Intel NUCs are defaulted to booting in UEFI mode, We need to change this.
On first boot,  press the F2 Key when you see the NUC logo, don't worry if you miss it just reboot the platform.
Under boot section of the GUI BIOS, disable or uncheck UEFI mode.  
Save and reboot, Press F10 to enter the boot menu and select the USB Key to boot from.
Follow the Ubuntu Install directions.  
Select Normal Install , Select Third-Party Drivers, on the Partition page select LVM.
Don't select Secure or you will have to enter a password to boot the platform.  
When prompted for User creation.  Let the install wizard use DHCP at first to configure the NIC we will change that later.

### Host Name

Use the following in configuring the hosts

>role=primary host, nodename=LABNUC01
>role=secondary host, nodename=LABNUC02
>username = labadmin
>password = Cisc0DN@

## Platform Update

Make sure the platform is up to date. From the linux command prompt run the following commands:

**$** ```sudo apt update```

**$** ```sudo apt list --upgradeable```

**$** ```sudo apt dist-upgrade```
  
## Configure Networking

We need to build a Bridge interface then the Virtual Machines will connect to the Bridge.
Use the linux command to display network interfaces on the system.
By default a process called **Network Manager** will be controlling the interfaces.  We need to move that control to **networkd** process.  
We can do this by redirecting the default netplan config to leverage networkd

### Disable NetworkManager

**$** ```sudo systemctl stop NetworkManager```

**$** ```sudo systemctl disable NetworkManager```

Run the command **ip address** to determine what the main ethernet interface is.  On my NUC it is **eno1**

### Configure Interface

**From the command prompt type:**

**$** ```ip address```

**Interface output**
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>ink/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 inet 127.0.0.1/8 scope host lo valid_lft forever preferred_lft >forever
>inet6 ::1/128 scope host valid_lft forever preferred_lft forever
>2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br0 state UP group default qlen 1000
>link/ether 94:c6:91:af:50:c5 brd ff:ff:ff:ff:ff:ff

#### Configure Netplan

Configure Netplan file .  Netplan uses .yaml file format  it loves indents.. use a graphical editor
that supports .yaml files it will help.   Sublime , or Visual Studio Code can assist.
So follow the included pattern correctly.

#### Apply network config

Two commands that go with Netplan are:

- netplan generate
- netplan apply

**netplan generate** will create the netplan config file , and check for errors. **netplan apply** will apply the configuration.   Like mentioned before we are going to create a bridged interface called **br0**
Compare to  **01-netconfg.yaml** located in the sample files folder /etc/netplan directory.

**From the command prompt:**

**$** ```netplan generate```

**$** ```netplan apply```

## Install Services

Install KVM,  ISC-DHCPd, lldp and Services

**$** ```sudo apt update```

### Core KVM Componants

**From the command prompt:**

**$** ```sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils```

**$** ```sudo apt install virt-manager```

### Network Services

**From the command prompt:**

**$** ```sudo apt install isc-dhcp-server```

**$** ```sudo apt install ntp```

**$** ```sudo apt install lldpd snmpd```

**$** ```sudo apt install bind9```

## Configure Services

Download  the services configuration tar file to the host **labnuc01.services.tar**  
unpack the tar file These are the base configuration files needed for the services
if you do this from the gui the files will be in the /home/labadmin/Downloads/ directory  

**From the command prompt:** move to user labadmin home directory.

**$** ```cd ~```

**$** ```tar -xvf ~/Downloads/labnuc0x.services.tar```

This should unpact the .tar file in the root of labadmin home directory. There shoudl be a new directory called labnuc01 for the first host and on second host unpack labnuc02.services.tar

### Configure DHCPD

Two key files in dhcp configuration , first is  /etc/default/isc-dhcp-server

This file gives us the interface to listen on.  See DHCP Config file.

Copy inclosed dhcp file to /etc/dhcpd/dhcpd.conf
Be sure to note difference between platform LABNUC01 and LABNUC02  

Copy isc-dhcp-server /etc/default/isc-dhcp-server
LABNUC01 will be Primary dhcp server LABNUC02 will be secondary

**$** ```sudo cp labnuc01/etc/dhcpd/dhcp.conf /etc/dhcpd/dhcp.conf```

**$** ```sudo cp labnuc01/etc/default/isc-dhcp-server /etc/default/isc-dhcp-server```

Edit the isc-dhcp-server file in /etc/default directory
use the following command from the command prompt:

**$** ```sudo nano /etc/default/isc-dhcp-server```

make sure the line : **INTERFACESv4="eno1,br0,vlan10"** reflects the interfaces you created using the **01-netplan-config.yaml file**.

#### DHCPd Restart

We need to restart the DHCPD service to pick up the new version of the dhcpd.conf file. To make sure there are no issues monitor the syslog.  A good way to do this is open two terminal sessions to the host, either from the desktop or ssh

**In Window one from the command prompt:**

**$** ```sudo tail -f /var/log/syslog | egrp "dhcpd"```

**In Window two from the command prompt:**

**$** ```sudo systemctl restart isc-dhcp-server```

**Watch the log file for messages like :**
>May 13 13:40:07 LABNUC01 dhcpd[21154]: Config file: /etc/dhcp/dhcpd.conf
>
>.
>May 13 13:40:08 LABNUC01 dhcpd[21154]: Listening on LPF/vlan10/b2:33:0e:bf:4b:69/10.10.10.0/24
>May 13 13:40:08 LABNUC01 dhcpd[21154]: Sending on   LPF/vlan10/b2:33:0e:bf:4b:69/10.10.10.0/24
>May 13 13:40:08 LABNUC01 dhcpd[21154]: Listening on LPF/br0/b2:33:0e:bf:4b:69/10.10.0.0/24
>May 13 13:40:08 LABNUC01 dhcpd[21154]: Sending on   LPF/br0/b2:33:0e:bf:4b:69/10.10.0.0/24
>.
>.
>May 13 13:40:08 LABNUC01 dhcpd[21154]: Server starting service.

### Configure_NTP

Copy the ntp.conf file to the directory /etc/
By default the ntpd service is not enabled or started.
First enable the ntp service with the systemctl enable command then start the service
Again with two terminal sessions one to watch the service start one to start the service.

**$** ```sudo cp labnuc01/etc/ntp.conf /etc/ntp.conf```

 **In Window one**

**$** ```sudo tail -f /var/log/syslog | egrp "ntpd"```

 **In Window two**

**$** ```sudo systemctl start ntp```
  
**Watch the log file for messages like:**
>May 13 13:51:12 LABNUC01 ntpd[21266]: Listen normally on 2 lo 127.0.0.1:123
>May 13 13:51:12 LABNUC01 ntpd[21266]: Listen normally on 3 br0 10.10.0.2:123
>May 13 13:51:12 LABNUC01 ntpd[21266]: Listen normally on 4 vlan10 10.10.10.2:123
>May 13 13:51:13 LABNUC01 ntpd[21266]: Soliciting pool server 45.33.103.94
>May 13 13:51:14 LABNUC01 ntpd[21266]: Soliciting pool server 108.61.73.244
>May 13 13:51:14 LABNUC01 ntpd[21266]: Soliciting pool server 47.190.36.230
>May 13 13:51:15 LABNUC01 ntpd[21266]: Soliciting pool server 173.255.140.30
>May 13 13:51:15 LABNUC01 ntpd[21266]: Soliciting pool server 142.147.92.5
>May 13 13:51:16 LABNUC01 ntpd[21266]: Soliciting pool server 185.144.157.134

### Configure DNS

Bind 9 DNS is used to provide name services for the lab.  Bind is only running on one node labnuc01 
to configure there are 5 main files we need to be concerned with to make sure we have a solid DNS resolution system. 
 you shouldn't need to make any changes to the files unless you change your host addressing structure.  
  
- named.conf - basic configuration
- named.conf.options - Options for interface to listen on
- named.conf.local - Pointers to zone files
- 10.10.db - reverse look up zone file
- 9800-lab.cisco.com.db - zone file

 In the /etc/bind directory make a new subdirectory called zones

**$** ```mkdir /etc/bind/zones```

Copy the files from the services tar labnuc01/etc/bind directory to the /etc/bind directory

**$** ```sudo cp -r labnuc01/etc/bind /etc/bind```

### Configure LLDP

     LLDP can be used to validate that the networking bridges are working ok, LLDPD supports both LLDP and CDPv1,V2.
        To configure lldpc copy the lldpd file from the  labnuc01/etc/default/ to /etc/default/ then restart the service
**$** ```sudo cp labnuc01/etc/default/lldpd /etc/default/lldpd```

**$** ```sudo systemctl enable lldpd```

**$** ```sudo systemctl start lldpd```

**Check the status of the process**

**$** ```sudo systemctl status lldpd```

**Watch the log file for messages like**  

>             lldpd.service - LLDP daemon
>                Loaded: loaded (/lib/systemd/system/lldpd.service; enabled; vendor preset: enabled)
>                Active: active (running) since Tue 2019-05-14 09:59:03 EDT; 6s ago
>                    Docs: man:lldpd(8)
>               Process: 8934 ExecStartPre=/bin/mkdir -p /var/run/lldpd (code=exited, status=0/SUCCESS)
>                Main PID: 8936 (lldpd)
>                    Tasks: 2 (limit: 4915)
>                CGroup: /system.slice/lldpd.service
>                        ├─8936 lldpd: monitor. 
>                        └─8938 lldpd: connected to c9800-POD08.
>
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol LLDP enabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol CDPv1 enabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol CDPv2 enabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol SONMP disabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol EDP disabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: protocol FDP disabled
>               May 14 09:59:03 LABNUC02 lldpd[8938]: libevent 2.1.8-stable initialized with epoll method
>               May 14 09:59:03 LABNUC02 lldpd[8938]: enable SNMP subagent
>               May 14 09:59:03 LABNUC02 lldpd[8938]: NET-SNMP version 5.7.3 AgentX subagent connected
>               May 14 09:59:03 LABNUC02 lldpcli[8937]: lldpd should resume operations
**Check for Neighbors**

**$** ```lldpcli show neighbors```

**Output should look like this**
>            LLDP neighbors:
>            -------------------------------------------------------------------------------
>            Interface:    br0, via: CDPv2, RID: 1, Time: 14 days, 12:54:19
>            Chassis:
>                ChassisID:    local C9300-24
>                SysName:      C9300-24
>                SysDescr:     cisco C9300-24P running on
>                           Cisco IOS Software [Fuji], Catalyst L3 Switch Software (CAT9K_IOSXE), Version 16.8.1s, RELEASE 
>           SOFTWARE (fc2)
>                            Technical Support: http://www.cisco.com/techsupport
>                            Copyright (c) 1986-2018 by Cisco Systems, Inc.
>                            Compiled Sat 26-May-18 23:31 by mcpre
>               MgmtIP:       10.10.0.254
>                Capability:   Bridge, on
>                Capability:   Router, on
>            Port:
>                PortID:       ifname GigabitEthernet1/0/3
>                PortDescr:    GigabitEthernet1/0/3
>                TTL:          180
  
### Configure KVM

KVM has multiple componants libvirt and qemu are the two most prominant that we will have to deal with.
home for the configuration files we need are in: /etc/libvirt directory structure.  
First we need to create a network that the pods will connect to.  We are going to leverage the bridge interface
"br0" created with netplan.  The network file resides in /etc/libvirt/qemu/networks/

First create a file called br0.xml in the /etc/libvirt/qemu/networks directory.  Use the br0.xml int the sample files folder as a template. you can also download and copy it to the directory.  
Then we need to tell libvirt and qemu about the new network , this is done by running a couple of virsh commands.
First Create the network instance.

**$** ```cd /etc/libvirt/qemu/networks```
Create the file br0.xml

**$** ```sudo virsh net-create br0.xml```

Then set the new network to auto start buy running :

**$** ```sudo virsh net-autostart br0```

<!-- NB Need to finish this section -->

Create KVM for 9800 / Download 9800 Code to platform
Create KVM for ISE , But don't start.

## ISE Setup

Create KVM for ISE , But don't start it you need to trick ISE into thinking it is on a Red Hat KVM instance. 
You will need to edit the ISE qemu .xml file and add a couple of sections in the Sample Files folder, ISE-KVM-Additions.xml is the instructions on what to add.  
you will need to change your working directory to the qemu directory. From the command prompt:

**$** ```cd /etc/libvirt/qemu```

Then edit the file using the libvirt program called virsh.  Substute the ISE below for what ever you called your VM Guest.  if you are un sure do directory listing, just note you do not need the .xml ending on the file.  

**$** ```virsh edit ISE```

Start the appliance, you may need to re-attach the ISO to the VM do this through the virt-manager in the GUI.  
Use the below parameters for your setup.

>host name: ise<br>
>IP Address: 10.10.0.20<br>
>Default Gateway: 10.10.0.254<br>
>ip domain-name: C9800-testdrive.us<br>
>ip domain-server : 10.10.0.2<br>
>ntp server: 10.10.0.2<br>

Once system is running from the Admin GUI, change the Administrative Password Settings.
GUI Navigation : **Adminstration/Admin Access/Password Policy**
Under Password History **un-check** password must be different
Under Password Lifetime **un-check** Administrators Password Expires

Import Device File (See Sample Files Directory)
Import User File  Key is **CiscoDNA!** (See Sample Files Directory)

## Appendix

### Referenced Documentation

[Cisco C9800-CL Virtual Controller configuration Guide](https://www.cisco.com/c/en/us/td/docs/wireless/controller/technotes/8-8/b_c9800_wireless_controller_virtual_dg.html)

### Linux Text Editor

I use nano as the text editor for the linux platform key commands:
> ^x - to save and  exit<br>
> ^w - to find<br>
> ^\ - to find and replace

If you are in ***"ios"*** mindset and hit a **^z** then all is not lost, from the command prompt type:

**$** ```fg```

### Check System Status

To monitor syslog

- **$** ```tail -f /var/log/syslog | egrep 'dhcpd|named|ntp'```

check status of indivitdual process

- **$** ```sudo systemctl status isc-dhcp-server```
- **$** ```sudo systemctl status ntp```
- **$** ```sudo systemctl status networking```

Check that the bridge is working

- **$** ```lldpcli show neighbors```

Check ip networking

- **$** ```ip address```
- **$** ```ip route```

To restart a daemon

- **$** ```sudo systemctl restart named```
- **$** ```sudo systemctl restart isc-dhcp-server```
- **$** ``sudo systemctl restart lldpd``

.... You get the Picture...
***EOF***