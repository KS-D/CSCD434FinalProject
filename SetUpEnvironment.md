# Environment Setup

## Install Virtual Machines

This lab will require 2 virtual machines. One Kali Linux machine and one Ubuntu 20.04 machine.  
These require 2 iso images. The windows 10 iso is optional, but beneficial because it allows you to use internet explorer which is a bit less secure than firefox or a chromium based web browser.

- kali-linux-2020.1b-installer-amd64.iso  
- ubuntu-20.04-desktop-amd64.iso  
- Windows10.iso

These two images will be installed in VirtualBox. The ram was set to 2gb for both. These will be running at the same time so select an amount of ram that your system can handle giving up.

## Network Setup

These virtual machines will need to be networked together to mimic them both being on the same local network. To do this you must go the Tools section of VirtualBox. Then select Preferences and select Network.  Select the green plus button to add a nat network. You can double click it to rename the network as well as define the IP addresses for the NAT. I set the IP to 192.168.1.0/24.  

Next you must select both the Kali linux virtual machine and select the settings. Change the `Attached to:` section to `Nat Network`, and then select the nat network you defined. Do the same for the Ubuntu virtual machine. **To confirm that you did this successfully run ip a in both virtual machines and then ping the other virtual machine.** They should report that the packets successfully travel to and from each virtual machine. If you have chosen to use a Windows.iso then you must also do this for the windows 10 machine. 

## Ettercap method

The first method for spoofing dns traffic will take place using ettercap in Kali. First you must figure out what the ip address of your Kali machine is. `ip a` will output the ip address in both linux systems. For windows `ipconfig` will output the ip address for a windows machine. Next you must determine what the default gateway is. It is likely the first ip address on your network i.e. 192.168.1.1. You can run `ip r` on a linux machine to confirm that is your default gateway or to determine what it is if you did not use the network described above. The next step to get this to work is to edit the /etc/ettercap/etter.conf file. You can use vim, nano, mousepad, etc. You will require sudo privileges to edit this file, and the other files we will be editing. This can be attained by either logging in as root or by using `sudo vim /etc/ettercap/etter.conf`. The first thing we will be editing in this file is the ec_uid, and ec_gid. You will replace the default values with 0. The reason for doing this is that setting these to 0 allows them to run as root. *include picture here* This allows the program to run with fewer restrictions. The final edit in this file will be removing the comments on the 2 lines below `# if using iptables`, as well as the area where the area under `# pendant for IPv6`. I do not believe that it is necessary, but otherwise ettercap will not run. *Include picture here*  

The next file we will be editing is the /etc/ettercap/etter.dns file. This file pertains to ettercap's DNS spoofing plugin. This is where you will place the domains you want to be directed to your kali machine. In this file there is a section that contains microsoft sucks ;). Below this comment is where you will place the domains you are trying to get to redirect to your machine. **Domains to avoid:** Websites like facebook, youtube, twitter, etc. will not work using this method. This is becuase modern browsers utilize *Don't remember what it is called will put here later*. A good initial website to use is penguin.ewu.edu since since it is http. Add that to the etter.dns file.

You will want to run the command `service apache2 start`. If you are not logged in as root you will need to add sudo to the beginning of that command. This is starting the web server that we are trying to redirect the victim to.

The next step is to start ettercap. When running this program you will want to be running as root to have the proper permissions to properly sniff network traffic. We will use the GUI version, so select that from the programs, or enter ettercap -G to start the program. Turn sniffing at start up off and then press the check to start the program. Click the options button (the one with 3 virtical circles). Select hosts and then select scan for hosts. Select ip address of the virtual machine you are trying to redirect to a fake website. Select the network gateway (likely x.x.x.1) and add that to target 2. From the options menu select  

Select the Man in the Middle menu (picture here). Select arp poisoning.

Now press the play button in ettercap. Go to your target machine and try to visit the penguin.ewu.edu webpage. You should see the Apache2 Debain default page. -- What can we do with this?

*Make a fake webpage for penguin with malicious downloads.*

