# Lab Shutdown Procedures 

## Power Down Pods on LABNUC02

Connect to each pod (05-08) via  ssh or through virt-manager ,

- copy ``flash://C9800-POD0X-Config startup-config``
- Shutdown the pod, using virt-manager
- After all pods are shutdown , either from linux prompt or from GUI shutdown the linux box.  ``sudo shutdown -h now``

## Power Down Pods LABNUC01

Connect to pod 01-04 via ssh or through virt-manager.  

- ``copy flash://C9800-POD0X-Config startup-config``
- shutdown the pod using virt-manager
- Connect to ISE vm - from the console run ``application stop ise``
- Validate ISE node processes are stoped ``show application status ise`` 
- Type from the  ise command line  ``halt``
- From virt-manager power down POD00
- after all VMs are shutdown either from linux prompt or from GUI shutdown the linux box. ``sudo shutdown -h now``

## Rest of Lab Kit

The rest of the lab kit can be powered off without specific intervention. 