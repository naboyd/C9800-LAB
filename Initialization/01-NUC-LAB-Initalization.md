# Host Initialization 

## Platform_Install

   <p>  Install OS- Ubuntu 18.04
        Download from ubuntu.com and burn on an ISO
        Ubuntu 18.04 Desktop
        https://www.ubuntu.com/download/desktop/thank-you?country=US&version=18.04.1&architecture=amd64
        Burn to USB Stick. I use:<br>
        Balena Etcher https://www.balena.io/etcher/
   </p>

   <p>  Boot platform from USB Stick and follow install instructions .
        The Intel NUCs are defaulted to booting in UEFI mode, We need to change this. 
        On first boot,  press the F2 Key when you see the NUC logo, don't worry if you miss it just reboot the platform. 
        Under boot section of the gui BIOS, disable or uncheck UEFI mode.  
        Save and reboot, Press F10 to enter the boot menu and select the USB Key to boot from.
        Follow the Ubuntu Install directions.  
        Select Normal Install , Select Third-Party Drivers, on the Partition page select LVM.
        Don't select Secure or you will have to enter a password to boot the platform.  
        When prompted for User creation.  Let the install wizzard use DHCP at first to configure the NIC we will change that later. 
  </p>

### Host Name

<p> Use the following in configuring the hosts<br></p> 

>role=primary host, nodename=LABNUC01<br>
>role=secondary host, nodename=LABNUC02<br>
>username = labadmin<br>
>password = Cisc0DN@<br>

## Plaform Update

<p> Make sure the platform is up to date from linux prompt</p>

<code> sudo apt update </code><br>
<code> sudo apt list --upgradeable </code><br>
<code> sudo apt dist-upgrade </code><br>
  
## Configure Networking

<p> We need to build a Bridge interface then the Virtuial Machienes will connect to the Bridge.
    Use the linux command to display network interfaces on the system.
    By default a process called Network Manager will be controling the interfaces.  We need to move that control to networkd process.  
    We can do this by redirecting the default netplan config to leverage networkd
</p>

### Disable NetworkManager

<code> sudo systemctl stop NetworkManager </code><br>
<code> sudo systemctl disable NetworkManager</code><br>
    
<p>Run the command <b>ip a</b> to determin what the main ethernet interface is.  On my NUC it is <b>"eno1"</b> 

### Configure Interface

**From the command prompt type:**
<code>ip a</code><br>

**Interface output**
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>ink/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 inet 127.0.0.1/8 scope host lo valid_lft forever preferred_lft >forever
>inet6 ::1/128 scope host valid_lft forever preferred_lft forever
>2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br0 state UP group default qlen 1000
>link/ether 94:c6:91:af:50:c5 brd ff:ff:ff:ff:ff:ff

#### Configure Netplan

<p> Configure Netplan file .  Netplan uses .yaml file format  it loves indents.. use a graphical editor
    that supports .yaml files it will help.   Sublime , or Visual Studio Code can assist.
    So follow the included pattern correctly.
</p>

#### Apply network config

<p> Two commands that go with Netplan are : </p>
 - netplan generate<br> 
 - netplan apply.<br>
<p> Generate will create the netplan config file , and check for errors.   netplan apply will apply the configuration.   Like mentioned before we are going to create a bridged interface called br0 
Compare to  01-netconfg.yaml 
</p>

**From the command prompt:** 
<code>   netplan generate</code><br>
<code>   netplan apply</code> <br>

## Install Services

<p>Install KVM,  ISC-DHCPd, lldp and Services </p>
<code>    sudo apt update<code><br>

### Core KVM Componants 

**From the command prompt:** 
    <code>sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils <br> 
    sudo apt install virt-manager </code>

### Network Services

**From the command prompt:** 
    <code>sudo apt install isc-dhcp-server<br>
    sudo apt install ntp<br>
    sudo apt install lldpd snmpd<br> 
    sudo apt install bind9</code><br>

## Configure Services

<p> Download  the services configuration tar file to the host <b>labnuc01.services.tar</b>  
    unpack the tar file These are the base configuration files needed for the services
    if you do this from the gui the files will be in the /home/labadmin/Downloads/ directory  
</p>

**From the command prompt:** 

<code> tar -xvf labnuc0x.services.tar</code>

### Configure DHCPD

<p>     Two key files in dhcp configuration , first is  /etc/default/isc-dhcp-server 
        This file gives us the interface to listen on.  See DHCP Config file. 
        Copy inclosed dhcp file to /etc/dhcpd/dhcpd.conf 
        Be sure to note difference between platform LABNUC01 and LABNUC02  
        Copy isc-dhcp-server /etc/default/isc-dhcp-server 
        LABNUC01 will be Primary dhcp server LABNUC02 will be secondary 
</p>
<code> sudo cp labnuc01/etc/dhcpd/dhcp.conf /etc/dhcpd/dhcp.conf 
       sudo cp labnuc01/etc/default/isc-dhcp-server /etc/default/isc-dhcp-server 
</code>
<p>         Edit the isc-dhcp-server file in /etc/default directory 
            make sure the line : INTERFACESv4="eno1,br0,vlan10" reflects the interfaces 
            you created using the 01-netplan-config.yaml file
</p>

#### DHCPd Restart

<p>      Restart the DHCPD service , Monitor the syslog to verify it is running,  open two terminal sessions to the host
         either from the desktop or ssh 
</p>

**In Window one** 
**From the command prompt:** 
<code>   sudo tail -f /var/log/syslog | egrp "dhcpd" </code>
**In Window two**
**From the command prompt:** 
<code>   sudo systemctl restart isc-dhcp-server</code> 
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

<p>      Copy the ntp.conf file to the directory /etc/ 
         By default the ntpd service is not enabled or started. 
         First enable the ntp service with the systemctl enable command then start the service
         Again with two terminal sessions one to watch the service start one to start the service. 
</p>
<code> sudo cp labnuc01/etc/ntp.conf /etc/ntp.conf </code>

 **In Window one** 
<code> sudo tail -f /var/log/syslog | egrp "ntpd" </code>

 **In Window two**
<code> sudo systemctl start ntp</code>
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

<p> Bind 9 DNS is used to provide name services for the lab.  Bind is only running on one node labnuc01 
     to configure there are 5 main files we need to be concerned with to make sure we have a solid DNS resolution system. 
     you shouldn't need to make any changes to the files unless you change your host addressing structure.  
    - named.conf - basic configuration 
    - named.conf.options - Options for interface to listen on 
    - named.conf.local - Pointers to zone files
    - 10.10.db - reverse look up zone file
    - 9800-lab.cisco.com.db - zone file
</p>
<p> In the /etc/bind directory make a new subdirectory called zones 
<code> mkdir /etc/bind/zones </code>
Copy the files from the services tar labnuc01/etc/bind directory to the /etc/bind directory
    $<code>sudo cp -r labnuc01/etc/bind /etc/bind  </code> 

### Configure LLDP

<p>     LLDP can be used to validate that the networking bridges are working ok, LLDPD supports both LLDP and CDPv1,V2.
        To configure lldpc copy the lldpd file from the  labnuc01/etc/default/ to /etc/default/ then restart the service
<code>  sudo cp labnuc01/etc/default/lldpd /etc/default/lldpd
        sudo systemctl enable lldpd 
        sudo systemctl start lldpd 
</code> 

**Check the status of the process**
<code>  sudo systemctl status lldpd </code>
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
<code>   lldpcli show neighbors</code>
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

        <!--KVM has multiple componants libvirt and qemu are the two most prominant that we will have to deal with. 
            home for the configuration files we need are in: /etc/libvirt directory structure.  
            First we need to create a network that the pods will connect to.  We are going to leverage the bridge interface 
            "br0" created with netplan.  The network file resides in /etc/libvirt/qemu/networks/ 
        --> 
        <Create_KVM_Network>
        
        </Create_KVM_Network>
Create KVM for 9800 / Download 9800 Code to platform
Create KVM for ISE , But don't start. 
See ISE-KVM-Additions.xml --> 

cd /etc/libvirt/qemu
labadmin@host:/etc/libvirt/qemu$ virsh edit ISE-2.1 // No .xml on the end.


// https://www.cisco.com/c/en/us/td/docs/wireless/controller/technotes/8-8/b_c9800_wireless_controller_virtual_dg.html
</Configure_Services>
// Check System Status
sudo systemctl status isc-dhcp-server
sudo systemctl status ntp
sudo systemctl status networking

<ISE>
    <ISE_Configuration>
    host name: ise 
    IP Address: 10.10.0.20
    Default Gateway: 10.10.0.254
    ip domain-name: C9800-testdrive.us 
    ip domain-server : 10.10.0.2 
    ntp server: 10.10.0.2
    </ISE_Configuration>
<!-- Once system is running from the Admin GUI, change the Administrative Password Settings , 
GUI Navigation : Adminstration/Admin Access/Password Policy 
Under Password History un-check password must be different 
Under Password Lifetime un-check Administrators Password Expires 
--> 

</ISE>
