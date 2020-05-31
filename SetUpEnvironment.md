# Environment Setup

## Install Virtual Machines

This lab will require 4 virtual machines. One Kali Linux machine and one Ubuntu 20.04 machine.  
These require 4 iso images. The windows 10 iso is optional, but beneficial because it allows you to use internet explorer which is a bit less secure than firefox or a chromium based web browser. You will be running at most 3 virtual machines at a time so keep that in mind when allocating resources for the virtual machines.  

- kali-linux-2020.1b-installer-amd64.iso  
- ubuntu-20.04-desktop-amd64.iso  
- any other virtual machine.

These images will be installed in VirtualBox. The ram was set to 2gb for both. These will be running at the same time so select an amount of ram that your system can handle giving up.

## Network Setup

These virtual machines will need to be networked together to mimic them both being on the same local network. To do this you must go the Tools section of VirtualBox. Then select Preferences and select Network.  Select the green plus button to add a nat network. You can double click it to rename the network as well as define the IP addresses for the NAT. I set the IP to 192.168.1.0/24.  

Next you must select both the Kali linux virtual machine and select the settings. Change the `Attached to:` section to `Nat Network`, and then select the nat network you defined. Do the same for the Ubuntu virtual machine. **To confirm that you did this successfully run ip a in both virtual machines and then ping the other virtual machine.** They should report that the packets successfully travel to and from each virtual machine. If you have chosen to use a Windows.iso then you must also do this for the windows 10 machine. 

## Ettercap

Ettercap is an open source program that allows the user to perform man in the middle attacks, as well as packet sniffing. We will look at 2 different methods of accomplishing this. The first will be a denial of service on a local network device. The second will be arp poisoning and dns spoofing. 

### Packet Sniffing

Ettercap can be used to passively sniff traffic on the local network. If someone on the network visits an http website anything of value, such as a username, password, or session cookie this will be output to the We will be using [testing-ground](http://testing-ground.scraping.pro/login) to illustrate this. Start ettercap with `sudo ettercap -G` in a terminal. Visit the testing ground url on you ubuntu machine and enter in whatever you want in both the username and password inputs and then hit login. Capture the output in ettercap.

### Denial of Service attack

You will want to be running 3 virtual machines for this section. The ubuntu 20.04 machine, the kali machine, and any third virtual machine on the same network. You will need to record the ip address of the ubuntu20.04 machine. Run `ip a` in a terminal. The IP address will beside the inet.

From the 3 machine, ensure that you can ssh to the ubuntu machine, as well as visit the webpage hosted on the ubuntu machine. 

Denial of service attacks work by flooding devices with packets. Ettercap has a plug in specifically for performing a denial of service. Generally when you open ettercap you will want to do it with root privileges. To open ettercap open a terminal and type `sudo ettercap -G` This opens the GUI version of ettercap. On in the start menu, the setup should remain default. You may choose to not sniff at start up. Select the ettercap menu, and then select plugins, and then select manage plugins. Then select dos_attack, enter in the IP address of the ubuntu machine, and then the enter any other ip address, but make sure to record the IP address you utilize. The first Ip address is the target to Dos and the second is the address that will be used as the source of the packets. 

Now the denial of service should be taking place. Try to ssh into the ubuntu machine. What is the output when attempting to ssh to the ubuntu machine? Now try to visit the website hosted on the ubuntu machine. What does your browser tell you when try to connect to the website?

Preventing this on the ubuntu machine: To prevent this we will be making a rule using uncomplicated firewall to make a rule to block these malicious packets. We will be using tcpdump on the ubuntu machine to inspect the ip address of these packets. Capture a portion of the ouput. Do you notice anything interesting in the output?  

Now we will create a rule to block the packets from the attack machine. To create a rule for uncomplicated firewall enter `sudo ufw deny from [The IP address you used in ettercap] to any`. Enter the IP address you used without the square brackets. To confirm that your rule was added run `sudo ufw status`. Now try to ssh or visit the website hosted on the ubuntu machine, You should be able to connect to the machine now.  

### DNS Spoofing with Arp poisoning

The first method for spoofing dns traffic will take place using ettercap in Kali. First you must figure out what the ip address of your Kali machine is. `ip a` will output the ip address in both linux systems. For windows `ipconfig` will output the ip address for a windows machine. Next you must determine what the default gateway is. It is likely the first ip address on your network i.e. 192.168.1.1. You can run `ip r` on a linux machine to confirm that is your default gateway or to determine what it is if you did not use the network described above. The next step to get this to work is to edit the /etc/ettercap/etter.conf file. You can use vim, nano, mousepad, etc. You will require sudo privileges to edit this file, and the other files we will be editing. This can be attained by either logging in as root or by using `sudo vim /etc/ettercap/etter.conf`. The first thing we will be editing in this file is the ec_uid, and ec_gid. You will replace the default values with 0. The reason for doing this is that setting these to 0 allows them to run as root. *include picture here* This allows the program to run with fewer restrictions. The final edit in this file will be removing the comments on the 2 lines below `# if using iptables`, as well as the area where the area under `# pendant for IPv6`. I do not believe that it is necessary, but otherwise ettercap will not run. *Include picture here*  

The next file we will be editing is the /etc/ettercap/etter.dns file. This file pertains to ettercap's DNS spoofing plugin. This is where you will place the domains you want to be directed to your kali machine. In this file there is a section that contains microsoft sucks ;). Below this comment is where you will place the domains you are trying to get to redirect to your machine. **Domains to avoid:** Websites like facebook, youtube, twitter, etc. will not work using this method. This is because modern browsers utilize *Don't remember what it is called will put here later*. A good initial website to use is penguin.ewu.edu since since it is http. Add that to the etter.dns file.

You will want to run the command `service apache2 start`. If you are not logged in as root you will need to add sudo to the beginning of that command. This is starting the web server that we are trying to redirect the victim to.

The next step is to start ettercap. When running this program you will want to be running as root to have the proper permissions to properly sniff network traffic. We will use the GUI version, so select that from the programs, or enter ettercap -G to start the program. Turn sniffing at start up off and then press the check to start the program. Click the options button (the one with 3 virtical circles). Select hosts and then select scan for hosts. Select ip address of the virtual machine you are trying to redirect to a fake website. Select the network gateway (likely x.x.x.1) and add that to target 2. From the options menu select  

Select the Man in the Middle menu (picture here). Select arp poisoning.

Now press the play button in ettercap. Go to your target machine and try to visit the penguin.ewu.edu webpage. You should see the Apache2 Debain default page. -- What can we do with this?

*Make a fake webpage for penguin with malicious downloads.*

