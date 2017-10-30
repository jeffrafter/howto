# Asus WL520GU

Much of this is based on the MightyOhm wifi radio project. 

## Building OpenWRT

OpenWRT is a great platform for running Linux on the router. Unfortunately building OpenWRT is difficult on the Mac and on Windows. There are a number of solutions that make it possible, but the easiest solution is to just use a virtual machine. To do this we'll use [Vagrant](http://docs.vagrantup.com/v2/getting-started/) which allows us to control our virtual machine from the command line.

### Getting setup

Install [Virtual Box](https://www.virtualbox.org). 

Install [Vagrant](http://docs.vagrantup.com/v2/installation/).

Create a new folder, I used `~/Projects/robots/openwrt` and go to that folder in Terminal. Once there you'll want to create a `Vagrantfile` to do this:

    vagrant init ubuntu/trusty64

Your `Vagrantfile` will have a lot of options, but the most basic is:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure(2) do |config|
      config.vm.box = "ubuntu/trusty64"
      config.vm.network "forwarded_port", guest: 80, host: 8080
    end

Note that we are specifying Ubuntu as the operating system and we and forwarding a port. This will help later when connecting packages. With that we can launch the virtual machine and ssh to it:

    vagrant up
    vagrant ssh
    
Once you are in the vagrant shell you are on your Ubuntu machine. Switch to the home folder of your virtual shell (don't use the shared `/vagrant` folder as it is mounted case-insensitively):

    cd ~/
    
### Getting the packages needed to build

In order to build OpenWRT you need some packages:

    sudo apt-get install gawk bison gcc

Or everything (?):

    sudo apt-get install subversion build-essential libncurses5-dev zlib1g-dev gawk bison gcc git ccache gettext libssl-dev xsltproc zip 
    
    
### Changing the ownership of /usr/local

For some reason the local user doesn't own the folder `/usr/local`. There is probably a good reason for this, but you'll need to be able to add tools there in the build process. To simplify, I changed the ownership:

    sudo chown -R vagrant:vagrant /usr/local/    
    
### Getting the source        

Most of the work I have done with OpenWRT uses the 8.09 version (Kamikaze). The 8.09.2 version can be downloaded here: [https://downloads.openwrt.org/kamikaze/8.09.2/](https://downloads.openwrt.org/kamikaze/8.09.2/). Technically OpenWRT has advanced and the version specified is obsolete (but it works without kernel errors on my target device so I like it).

    svn co svn://svn.openwrt.org/openwrt/tags/8.09.2 openwrt
    
Or:

    wget https://downloads.openwrt.org/kamikaze/8.09.2/kamikaze_8.09.2_source.tar.bz2
    bzip2 -dk kamikaze_8.09.2_source.tar.bz2
    tar -xvf kamikaze_8.09.2_source.tar
    mv 8.09.2 openwrt     

Now that you've go the source go to it:

    cd openwrt
    ./scripts/feeds update -a
    ./scripts/feeds install madplay mpc mpd
    make prereq
    make menuconfig

#### Menu config

* Target System (Broadcom BCM947xx/953xx [2.4])
* Target Profile (Generic, Broadcom WiFi (default))
* Select all packages by default
	- Image configuration —>
		- Base system 
			- busybox (press enter to open hidden menu)
				- Configuration
					- Coreutils
						- stty
					- Networking Utilities
						- Netcat server options (-l) 
	- Kernel modules
		- Sound support
			- kmod-sound-core
		- USB support
			- kmod-usb-core
			- kmod-usb-ohci
			- kmod-usb-audio
		- Sound
			- mpd
			- mpc
			- madplay


#### Patch bug

Unfortunately the version of the `patch` tool on ubuntu has changed just slightly and the version string that is printed now has the word _GNU_. The easiest way to fix the problem is to adjust the the configure script for Quilt (where the error shows up).

	vim ./build_dir/host/quilt-0.47/configure.ac

Find the line:

	patch_version=$2
	
And change it to 

	patch_version=$3	
	
Save the file, then re-run the configure line:

	cd ./build_dir/host/quilt-0.47
	./configure --with-patch=/usr/bin/patch
	
	
### Missing packages bug

One of the package repositories (luci) moved to git and is no longer available at the previous location. If you are going to be working on packages you'll need to update this in the `feeds.conf.default` file:

    nano feeds.conf.default
    
Replace the luci line ([https://dev.openwrt.org/changeset/40030](https://dev.openwrt.org/changeset/40030))    

    src-svn packages svn://svn.openwrt.org/openwrt/branches/packages_8.09 svn://svn.openwrt.org/openwrt/packages
    src-svn xwrt http://x-wrt.googlecode.com/svn/tags/kamikaze_8.09.2/package
    # src-svn luci http://svn.luci.subsignal.org/luci/tags/0.8.8/contrib/package
    src-git luci http://git.openwrt.org/project/luci.git
	
Once complete, update the packages:

     ./scripts/feeds update	          
	
### Make the OpenWRT binary

At this point your configuration should be ready:

    make world V=99
    
The `V=99` adds a serious amount of debugging information. The compile should run and complete with no errors. This process takes a long time (about 30 minutes), so call your mom and watch the logs messages scroll by.    

Once this is completed you should copy the build to the shared `/vagrant` folder on the virtual machine. This will make it accessible from your laptop without an SSH session:

    cp bin/openwrt-brcm-2.4-squashfs.trx /vagrant
	
## Flashing

Now that you have built a custom OpenWRT you need to _flash_ it to the ROM of the router. Then you'll need to configure the network and possibly setup the banner message (for fun).

### Blind flashing

There are a couple of ways to flash the build onto the ROM. The easiest way is to flash it via the network cable. Unfortunately there is no way to see the output of this process unless you manually solder the header for the serial inside the unit. The alternative is to "Blind flash" the unit. You could end up making a mistake and bricking the router. But I haven't done that yet.

To accomplish a blind flash you'll need to be able to connect your computer directly to the router via a CAT-5 cable (not a crossover). Connect the cable from local machine (i.e., laptop) to LAN1 port on the router and open the network preferences for your LAN:

[https://rpl.cat/ZUEZX2pBV_aU8IfDO5mJtr5Fm_XlJIhm5MTsubrHxzw](https://rpl.cat/ZUEZX2pBV_aU8IfDO5mJtr5Fm_XlJIhm5MTsubrHxzw)

> Sometimes your wireless network (wireless) is on a .1 network and will interfere If so you can (a) change it (b) turn off wireless while you do stuff

Setup your LAN network: 

- Configure IPv4: *Manually* 
- IP Address: *192.168.1.185*
- Subnet Mask: *255.255.255.0* 
- Router *192.168.1.1*

You are ready to flash the unit. You need to get the unit into restore mode:

1. Power off router (unplug it)
2. Hold down Restore button on the back of the router
3. Power on router while still holding the button (plug it back in)
4. Power LED should start flashing once per second, release Restore button

At this point the router is in restore mode. You should be able to ping the router:

     ping 192.168.1.1
     
You should see valid ping responses:

	PING 192.168.1.1 (192.168.1.1): 56 data bytes
	64 bytes from 192.168.1.1: icmp_seq=0 ttl=100 time=2.480 ms
	64 bytes from 192.168.1.1: icmp_seq=1 ttl=100 time=1.561 ms
	64 bytes from 192.168.1.1: icmp_seq=2 ttl=100 time=1.462 ms
	64 bytes from 192.168.1.1: icmp_seq=3 ttl=100 time=1.212 ms
    
Change to your project folder (where the `Vagrantfile` is located, not in your SSH session):
     
    cd ~/Projects/robots/openwrt/  
    
Next start a `tftp` session:
    
    tftp
    > trace
    > timeout 1
    > mode binary
    > connect 192.168.1.1
    > put openwrt-brcm-2.4-squashfs.trx

You may see a lot of "Sending packet" kind of information then it will stop (for me this ended with `Sent 1904640 bytes in 5.6 seconds`).

<span style="color:red">*** Wait 10 minutes, don't touch anything ***</span>

*** During this point the router is reconfiguring the firmware you just uploaded ***
After 10 minutes, reboot the router (unplug, wait a couple seconds, plug it back in)
Ping the router (wait until it starts responding, boot takes about 10 seconds):

    ping 192.168.1.1 

Once it is responding you are ready to login:

    telnet 192.168.1.1	
    
You should see your router:/Users/njero/Desktop/Robots/robot/kamikaze/package/phidget21-servo-utils/Makefile

    Trying 192.168.1.1...
    Connected to 192.168.1.1.
    Escape character is '^]'.
     === IMPORTANT ============================
      Use 'passwd' to set your login password
      this will disable telnet and enable SSH
     ------------------------------------------


    BusyBox v1.11.2 (2015-08-31 04:19:00 UTC) built-in shell (ash)
    Enter 'help' for a list of built-in commands.

      _______                     ________        __
     |       |.-----.-----.-----.|  |  |  |.----.|  |_
     |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
     |_______||   __|_____|__|__||________||__|  |____|
              |__| W I R E L E S S   F R E E D O M
     KAMIKAZE (8.09.2, r18961) -------------------------
      * 10 oz Vodka       Shake well with ice and strain
      * 10 oz Triple sec  mixture into 10 shot glasses.
      * 10 oz lime juice  Salute!
     ---------------------------------------------------
    root@OpenWrt:/# exit
    
### SSH

To connect to the router via SSH you'll need to set a root password:

    passwd    
    	
## Networking

Once you've flashed OpenWRT onto your router, you'll want to configure the router to talk with your home network. Telnet to the router:

    telnet 192.168.1.1
    
### Wireless

To setup the wireless edit `/etc/config/wireless`. By default the configuration is:

    config wifi-device  wl0
      option type     broadcom
      option channel  5

      # REMOVE THIS LINE TO ENABLE WIFI:
      option disabled 1

    config wifi-iface
      option device   wl0
      option network  lan
      option mode     ap
      option ssid     OpenWrt
      option encryption none


The default configuration allows you to connect to the router as if it were your primary access point. However, in our configuration we want it to join our existing wireless netork. To do this change the configuration to the following:

    config wifi-device  wl0
      option type       broadcom
      option channel    6  # the channel your wireless network is on

    config wifi-iface
      option device     wl0
      option network    lan
      option mode       sta 
      option ssid       MyNetwork # the SSID of your network
      option encryption psk  # the encryption mode of your network (or wep)
      
      ### If you are using WPA preshared key, specify that key or password
      option key        XXXXXXXXXX 

      ### If you have a wep key with a password, you must prefix the password
      ### with "s:" (no quotes). If you are using a wep hex key then you should
      ### specify the key and then select it
      # option key1="ABCDEF12345...""
      # option key=1
       

> To get your wifi channel on a Mac, Option+Click the wifi icon in the menu bar and choose "Open Wireless Diagnostics". Once opened, hit Cmd+2 to open a general info panel. The channel is there.

### Network

We'll want to configure the network so that our device is serving as a bridge on the wireless network. To do this you have to update the lan configuration and the an configuration in `/etc/config/network`:


    #### VLAN configuration
    config switch eth0
      option vlan0    "1 2 3 4 5*"
      option vlan1    "0 5"


    #### Loopback configuration
    config interface loopback
      option ifname   "lo"
      option proto    static
      option ipaddr   127.0.0.1
      option netmask  255.0.0.0


    #### LAN configuration
    config interface lan
      option type     bridge
      option ifname   "eth0.0"
      option proto    dhcp # or static
      #### If you don't choose a dynamic ipaddr your wireless network may allow a static one
      #### but make sure it is on your wireless subnet. Choosing a static address makes
      #### connecting to your router simpler
      # option ipaddr   192.168.0.199
      # option netmask  255.255.255.0


    #### WAN configuration
    config interface wan
      option type     bridge  
      option ifname   "eth0.1"        
      option proto    static                
      option ipaddr   192.168.1.1 # Allow it to continue to operate as a gateway
      option netmask  255.255.255.0    

### DHCP

If you are using DHCP on WAN you will need to configure it in `/etc/config/dhcp` (notice that by default the WAN configuration is disabled):

    config dnsmasq
      option domainneeded     1
      option boguspriv        1
      option filterwin2k      '0'  #enable for dial on demand
      option localise_queries 1
      option local            '/lan/'
      option domain           'lan'
      option expandhosts      1
      option nonegcache       0
      option authoritative    1
      option readethers       1
      option leasefile        '/tmp/dhcp.leases'
      option resolvfile       '/tmp/resolv.conf.auto'
      #list server            '/mycompany.local/1.2.3.4'
      #option nonwildcard     0
      #list interface         br-lan

    config dhcp lan
      option interface        lan
      option start            100
      option limit            150
      option leasetime        12h

    config dhcp wan
      option interface        wan
      option start            100        
      option limit            150                
      option leasetime        12h            

### Firewall

By default the router is expecting to have the WAN port connected to the Internet so the firewall is settings are very restrictive. However because we will use the WAN port to directly connect to a laptop we'll want to remove some of the restrictions.

Open `/etc/config/firewall` and update the `wan` zone so that it is similar to the `lan` zone:

    #### Update the wan zone so it looks like the lan zone
    config zone
      option name     wan
      option input    ACCEPT
      option output   ACCEPT
      option forward  REJECT

### Applying changes

Once you have made the changes you can restart your network :

    /etc/init.d/network restart

This will disconnect you from your current session (and depending on the changes you make, you may need to reboot the router). Change the network cable to be plugged into the WAN port on the router and you can again telnet to `192.168.1.1`. You can also telnet through your wireless network by finding out which dhcp address it picked up (check your normal wireless router for clients), or by connecting to the manual address you specified (in our example `192.168.0.199`).
	
## Robotics and Phidgets package

First you need to install the package into the feeds for building packages:

    # In the main ssh session    
    cd ~/
    git clone git://github.com/jeffrafter/phidget-servo-driver
    mkdir openwrt/package/phidget-servo
    cp phidget-servo-driver/package/Makefile openwrt/package/phidget-servo-driver
    cd openwrt
    ./scripts/feeds install phidget-servo # not sure this is necessary

It will need to be accessible via http:

    # In another ssh session
    cd ~/phidget-servo-driver
    tar -cvf phidget-servo.tar ./src/*
    sudo python -m SimpleHTTPServer 80

Then you need to select it

    # In the main ssh session    
    make menuconfig
    # Go to libraries, select it with M, exit, save
    make package/phidget-servo/{clean,compile} V=99

When working on new things in the tar if you have already downloaded, you need to remove the cached download:

    rm /home/vagrant/openwrt/dl/phidget-servo.tar

At this point you have a compiled package (ipk). If you changed the `menuconfig` you'll need to recompile the primary binary:

    make world V=99

If you haven't changed the selected packages (just updated the package) you should be able to simply update the index:

    make package/index

You'll want to re-flash the binary to the router (see Flashing above). Finally, you'll need to make the new set of packages available to `opkg` so you'll need to copy them to where they can be seen from the router. 

## Opkg

The package tool for OpenWRT is `opkg`. By default the configuration (in `/etc/opkg.conf`) points to the default OpenWRT packages:

    src/gz snapshots http://downloads.openwrt.org/kamikaze/8.09.2/brcm-2.4/packages
    dest root /
    dest ram /tmp
    lists_dir ext /var/opkg-lists
    option overlay_root /jffs

What we need to do is expose the packages from our Virtual Machine to the router. This takes a couple of steps. First we need to run the python web server in the virtual machine:

     # From within your ~/openwrt folder
     sudo python -m SimpleHTTPServer 80
     
Once the server is running in the virtual machine, you should be able to connect to it via the port forward we setup in the `Vagrantfile`. From your local machine:

     wget http://127.0.0.1:8080/README

This should successfully get the `README`. With that accomplished you should be able to check your local IP address and connect directly to it from the router. On the router run:

    wget http://192.168.0.2:8080/README
    
If this fails (and you cannot ping your machine from the router) it is possibly being blocked by your local firewall. Temporarily turn off the firewall or add an exception for port 8080. 

Now that you are successfully connected, on the router, update the `/etc/opkg.conf` to point at your local machine and proxy:


    src/gz snapshots http://192.168.0.2:8080/bin/packages/mipsel                          
    dest root /
    dest ram /tmp
    lists_dir ext /var/opkg-lists                                   
    option overlay_root /jffs

Now that the packages are connected, update them:

    opkg update
    
Then install the `phidget-servo` package:    

    opkg install kmod-usb-ohci
    opkg install phidget-servo
    
Make sure the USB is disconnected and reboot the router. Once you have connected, without plugging in the USB, detect all of the devices and clear:

    dmesg -c
    
Now plug in the USB for the servo and run `dmesg` again:

    dmesg
    
You should see the information about the device

    hub.c: new USB device 00:03.0-1, assigned address 2
    usb.c: USB device 2 (vend/prod 0x6c2/0x39) is not claimed by any active driver.           
    
You should now be able to control the servo:

    servo 10 # servo min pos: -23, max pos: 232

## Resources 

[Setting up packages](https://forum.openwrt.org/viewtopic.php?id=16040)

[FCC versus OpenWRT](http://www.cnx-software.com/2015/07/27/new-fcc-rules-may-prevent-installing-openwrt-on-wifi-routers/)

[Building OpenWRT on Ubuntu and OSX](http://www.acme-dot.com/building-openwrt-14-07-barrier-breaker-on-ubuntu-and-os-x/)

[Magic potion](https://gist.github.com/jeffrafter/386741)
