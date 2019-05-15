# Lab Start Up Procedures

Procedure guid to bring up the lab environment.  

## Switch

1. Power up Ethernet Switch and Check console to make sure its up, and ready. 

## ASA

 Connect Port and power up ASA

1. Connect ASA Port 1 to UPLINK (Internet Access) this will be a conference room or host location ethernet port. It will do DHCP. 
2. Connect ASA Port 2 to Cat9300 Switch Port 1
3. Connect ASA Port 3 to Cat9300 Switch Port 2
4. Verify via console the ASA has powered up correctly. Verify the ASA has an external IP Address. 

## Hosts

Connect and power up the Two NUCs. , Do NUC01 First then NUC02 

1. Connect NUC 1 Ethernet to Cat9300 Switch Port 3
2. Run services checks on the NUC01. 
	1. ``sudo systemctl status ntp``
  	2. ``sudo systemctl status isc-dhcp-server``
  	3. ``sudo systemctl status bind9``
    4. ``ping 10.10.10.2`` ``ping 10.10.10.254`` ``ping 10.10.0.1``
3.  verify DNS is working by pinging Google.com (Does it resolve? )
4. Repeat Step 1- 3 for NUC2 

## Start VMs 

1. Start ISE VM Node
2. Login to ISE and verify system is up	```sho application status ise``` 
3. Bring POD00 up  Verify you can connect to @POD00_dot1x.  SSID this will validate ISE is working.
4. bring up pods 1-8
5. Reset each pod config, 
   1. ```copy flash:/C9800-POD0X-Config startup-config```
   2. ``reload system``  but do not save....

## Connect APs 

Connect 1 Catalyst 9100 AP and one Aironet Wave 2 AP to each of the PODs AP Ports.