![Minecraft Logo](https://minecraft.net/images/logo.png "Setting up a minecraft server")

# Setting up a Minecraft server

## Table of contents

* Minecraft 
* Clients and Servers
* Getting a server on the Internet
* Setting up the operating system
* Securing your server
    * Secure Shells
    * Firewall and iptables
    * fail2ban
    
* Configuring your server
  * Whitelisting
  * CraftBukkit
  * Protect your world
* MinecraftEdu
* Resources


## Minecraft

[Minecraft](https://minecraft.net) is a game by Mojang that allows players to mine, build and craft things in their own worlds. You can play Minecraft on your own computer, on a mobile device and on your Xbox. You don't need to connect to a server to play Minecraft, but a server lets you play with your friends.

## Clients and Servers

When computer programs need to communicate, they are often separated into client programs and server programs. Client programs run on user machines and server programs run on servers. Any computer can be a client or a server, but often servers are out on the Internet. This allows multiple clients to talk with them at the same time.

Servers are like cooks at a restaurant and clients are like customers. The customers send messages (via the waiter or waitress) to the cook who prepares a response (the food) which is sent back to the customer. One cook can serve a lot of customers at the same time but there are limits.

In software, clients connect to servers by specifying their IP address (or website) and sending messages on a specific port. Ports are like channels on your TV. On most computers the lower number ports are reserved for specific things (usually port 0-1023). Higher number ports can be used by anyone and sometimes conflict so make sure you are using the right port for your client and server. 

Servers and clients communicate by sending small messages back and forth. This usually happens using a *protocol* or agreement about how they will talk (who talks first and what happens if you don't hear part of the message). Most protocols are based on TCP or UDP. 

> [TCP](http://en.wikipedia.org/wiki/Transmission_Control_Protocol) is short for Transmission Control Protocol. It is one of the primary ways devices communicate on the Internet. Every time you go to a website, your web browser (a **client**) is connecting to a website (a web **server**) using a protocol called HTTP which is based on TCP. Messages are sent from your browser to the server and back using port 80 or port 443. The thing that makes TCP so great is that communication sockets are constantly interrupted (the Internet is always breaking). TCP guarantees that your messages will be received and that they will be in order. Without this, the hundreds of messages the server sends to your client might take different pathways and arrive out of order. Web pages would be all jumbled.

> [UDP](http://en.wikipedia.org/wiki/User_Datagram_Protocol) is short for User Datagram Protocol. UDP is also used on the web but is more commonly used in communications or video. For instance, Youtube uses UDP to deliver video to your web browser. And Skype uses UDP when you have a conversation (most of the time). UDP does not guarantee that your message will ever arrive, and if the order messed up... UDP won't care. This seems bad, but in the case of phone conversations means that there is no delay when waiting for the whole message. It skips missing packets of information and moves on.

Imagine you are playing the piano in a concert and the song you are playing is very fast and beautiful, but also very difficult. There is one note that you have to stretch your pinky to reach and sometimes you miss it. In the TCP version of this story, if you miss the note you stop the concert and keep trying to hit it with your pinky until you get it-- because playing all of the notes exactly as they are written is really important. In the UDP version of this story you just keep playing the song if you miss the note... very few people will notice and the whole thing will sound better if you don't stop.

Minecraft uses TCP port 25565 to communicate between the client and server. This means that the messages are reliably sent between the two (or there is a disconnect). 

## Getting a server on the Internet

Getting a server on the Internet is easy. In fact, your computer is connected to the Internet right now and could be a Minecraft server. In most cases, your router and cable modem are in between your computer and the Internet. This means that people outside of your network can't see your computer directly (in most cases), instead they see your cable modem or your router. For instance go to [WhatIsMyIP?](http://whatismyip.com) and see what your address is (it might be something like 23.240.3.21). If you were hosting a Minecraft server on your computer you would need to give your IP address to your friends. Unfortunately they still wouldn't see your computer, instead they would be connecting to your cable modem! 

You could use port forwarding to send any messages from your cable modem to your computer to solve this problem. In order to do that, you need to know what the address of your computer on your local network is. There are few ways to do this, but the best way is to open a terminal window (on OSX you can use Spotlight and search for Terminal) and run `ifconfig`. This is going to give you a lot of information–way more than you need–but what you are looking for is your `inet` address which usually starts with `192.` or `10.`. To weed this out you can use the more advanced command:

    ifconfig | grep inet
    
This command says "run `ifconfig` and send the output to the `grep` command and search for the word 'inet' and just show me those lines". With this information you could forward external messages to your computer on your internal network. Even if you did this though, your Minecraft server would only work while your computer is on and while your Internet is working. If someone on your network starts watching YouTube videos or a movie on Netflix, your bandwidth might not be enough to support playing Minecraft. Also, if you are running a server on the same machine you play minecraft, your computer might get slow. For these reasons, we'll use a different computer out on the Internet. 

Though there are some free computers you can use on the Internet, this is generally not the case (although Amazon offers 1 year free use of small servers). Computers cost money and running them costs money. All you have to do is choose the right server for running Minecraft without spending too much money.

For our use, we want to be able to play minecraft with 6 of our friends. A quick way to determine if the server you found will work is to go to [Can I host a minecraft server](http://canihostaminecraftserver.com/). We plan on using [DigitalOcean](https://www.digitalocean.com/?refcode=21447a91b37b) to host our server. DigitalOcean is a hosting company that lets you rent computers on the Internet (or rather, they let you rent *virtual* servers). DigitalOcean, like most providers, offers Tier 1 bandwidth at 1gbit/sec so we specify that our upload and download (in mbit/s) are **1024**. The smallest plan on [DigitalOcean](https://www.digitalocean.com/?refcode=21447a91b37b) is 5$ per month and it offers 512MB of memory (this can also be billed hourly if you want to turn off you machine to save money). According to *Can I host a minecraft server* that gives us just enough for 6 friends and us. 

There are lots of other providers (for example [Linode](https://www.linode.com/?r=ff8e1eabda7e9d9b57705dbb9533844c790aa02d), [Prgrmr](http://prgmr.com/xen/plans.html), [Amazon EC2](http://aws.amazon.com), [Rackspace](http://rackspace.com)).

### Getting started at Digital Ocean

Once you signup you'll need to setup your billing information. 

#### Create a Droplet

After that you will want to create a "Droplet". Think of this like a recipe for creating a computer of a specific size and with a specific operating system.

> A note on hostnames: when creating a droplet you have to create a hostname. People choose all kinds of names but when you choose your name try to imagine you will have lots of servers and you want to be able to distinguish them quickly. For this reason most people choose a naming theme to identify their servers quickly. For me, I use ancient city names but some people use articles of clothing or names of characters in books. You can come up with your own or you can choose something like "minecraftserver1". 

Choose a region that is near you or stick with the default. 

We'll be running Linux on our server. There are lots of tradeoffs to the different flavors of linux. For us we will choose the one that will make our job easier and because Ubuntu has so much information online and a large helpful community we'll go with that. Although there are newer versions we chose Ubuntu 12.04.3 x64. This is an "LTS" release which means it has Long Term Support. Any security fixes will be ported to this version and it will be stable for a long time.

For now we didn't choose to enable backups. We might regret that later.

Once you have finished configuring your server you can "Create Droplet" and DigitalOcean will start creating the machine and send you an email with login details.

#### SSH access to your new server

Using the information that DigitalOcean sent us we can create a "Secure Shell" or SSH connection to our server. This allows us to run commands on our remote server as if we were on it (and it encrypts all of the information being sent back and forth). 

First, you'll want setup a hostname for your remote server so you can login to the server by name. Open Terminal and type:

    sudo nano /etc/hosts
    
This opens the file `/etc/hosts` in the terminal editor `nano` as the admin ("sudo" means super-user do). At the top of the file add your server IP address followed by the hostname you chose. For example:

    123.456.789.123 yourhostname
    
Once complete, hit `Ctrl+X` to exit (not `CMD+X`). `nano` will ask if you want to save, press `Y` and hit enter. Now you can test to make sure that worked by "pinging" your server:

    ping yourhostname
 
You should see:

    PING yourhostname (74.125.224.71): 56 data bytes
    64 bytes from 74.125.224.71: icmp_seq=0 ttl=53 time=40.291 ms
    64 bytes from 74.125.224.71: icmp_seq=1 ttl=53 time=38.830 ms
    64 bytes from 74.125.224.71: icmp_seq=2 ttl=54 time=44.167 ms
    ...

Hit `Ctrl+c` to stop ping.
    
Now we can ssh in. Type:

    ssh root@yourhostname
    
You should see:

    The authenticity of host '123.456.789.123 (123.456.789.123)' can't be established.
    RSA key fingerprint is a1:c6:9f:b5:f9:bb:11:1a:a1:c6:9f:b5:f9:bb:11:1a.
    Are you sure you want to continue connecting (yes/no)? 

Type "yes" and hit enter. At this point you need to enter the password that you were sent in the email. Once you've entered that you are logged into your server on the Internet.

> Note: you just logged into your server as the `root` user. This is considered very bad practice and you want to avoid doing this. In fact you don't want anyone logging in as the root user because it is not secure. Without setting up a custom droplet though you need to do this once.


#### Securing your server

Much of this comes from [My First 5 Minutes On A Server; Or, Essential Security for Linux Servers](http://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers).

You can change your root password right away, but you need to store it somewhere. You could store it in your Keychain. It should be long and complex, you shouldn't need it.

    passwd
    
By default your server is set to use the most recent set of packages that server company has. But there are usually a lot of updates that you should start with:

    apt-get update
    apt-get upgrade
    
Once you have this we'll want to lock things down. If people try to hack into our server we want them to get blocked permanently.

    apt-get install fail2ban
    
##### Creating a user
   
Now we want a user. We'll call this user `minecraft`:

    useradd -D -s /bin/bash
    useradd minecraft
    mkdir /home/minecraft
    mkdir /home/minecraft/.ssh
    chmod 700 /home/minecraft/.ssh
 
Now that you have a user you want to setup an authorized key saying that your computer is allowed to login to the server without using a password. This is actually much safer than using a password because passwords can be guessed. Keys are much longer and encrypt all of you communications between your machine and the server. 

    nano /home/minecraft/.ssh/authorized_keys
    
The `nano` editor will open and you need to get some information from your local computer. Open another tab or another terminal window and in the new Terminal type:

    cat ~/.ssh/id_rsa.pub
    
You should see something like:

    $ cat ~/.ssh/id_rsa.pub 
    ssh-rsa ssflsjdjsfljslkdfjyUvMIvhJUYs7nIaiLBnUcs03XuOeHiw1JGh1M/ovbKc9YO4SJsl9CYxpyDeh9jSyvdNhNdeUSg7PBSSyAYpVXeK6WXN9LnqKOWRu8n5rXGNSycM2tenaADiS/xtMkHmIFYOE/QFQF+AmgdklfjsdfjlskjdflksjfdlkjslkjsIP/gxSsEDzOGSIxYqJwpCzV+/nwub72ElzVcW9EJu2HJHGYIUHK80980989089089089090OyFubRb7+V6Db4xc+x2HirdyMW7hZceDfTpB0xX1GXd4PYZ023gfvNkGbIj/aOSDeJtDIJyQhzCbIXY+CiLkU556e0rg2Nw== yourname@yourcomputer.local
 
Note that this is your *public* key. You also have a *private* key which you should not give away ever (it is in the file id_rsa without the `.pub` extension). 

Copy the contents (including `ssh-rsa` and `yourname@yourcomputer.local`) and paste it back in the `nano` window in the other terminal. Once pasted, click `Ctrl+X` type "Y" for "yes" and hit enter.

##### Super user access

Now we need to lock down that file so nobody can change it (and add their own keys). Once we have changed the mode so it is only readable, we will change the owner to our new `minecraft` user:

    chmod 400 /home/minecraft/.ssh/authorized_keys
    chown minecraft:minecraft /home/minecraft -R
    
We'll give the user a password which we will need when doing important things.

    passwd
    
Next we need to change how the `sudo` command works. The `sudo` command is short for "super-user do" and lets users run commands as if they were the `root` (most powerful) user. To change this we'll use a command called `visudo`. This command opens an editor called `vi` which works a little differently than the editors you are used to. Instead we'll force it to use `nano`:

    EDITOR=nano visudo
    
Once open you should see a lot of commands and you want to comment some of them out:

    %admin ALL=(ALL) ALL

and

    %sudo ALL=(ALL) ALL
    
We are going to comment these lines out by adding a "# " to the start of the lines:

    # %admin ALL=(ALL) ALL

and

    # %sudo ALL=(ALL) ALL
    
Next we'll add the line:

    minecraft ALL=(ALL) ALL
  
This says, as long as we are logged in and know the password we can run ALL commands as the superuser.
    
> Technically we shouldn't need this for our minecraft server as everything will be setup. Giving users `sudo` is very serious. If they have super user privileges they can change anything about the server. Sometimes when setting up a server you'll see people use `NOPASSWD:ALL` indicating that you don't even need a password. This is very dangerous and is usually just a sign of laziness. For us, it is good to have some access so that we can switch to the super user later.

##### Securing SSH

In general you don't want users using `ssh` to login as the root user (even though you are doing it right now).

*******

    nano /etc/ssh/sshd_config

Change PermitRootLogin from "yes" to "no"

    PermitRootLogin no
    
Change `PasswordAuthentication` to `no` and remove the `#` at the start of the line.
    
    PasswordAuthentication no

Do this

    UseDNS no
    
Add this line
 
    AllowUsers minecraft@(your-ip) minecraft@(another-ip-if-any)
 
Or

    AllowUsers minecraft
 
 Restart ssh:
 
    service ssh restart
    
*******


##### Blocking everything with a Firewall

*******

    ufw allow from YOURIP to any port 22
    ufw allow 25565
    ufw enable

You should allow this to happen once done

*******


   apt-get install unattended-upgrades

   nano /etc/apt/apt.conf.d/10periodic

Update the file to look like this

    APT::Periodic::Update-Package-Lists "1";
    APT::Periodic::Download-Upgradeable-Packages "1";
    APT::Periodic::AutocleanInterval "7";
    APT::Periodic::Unattended-Upgrade "1";


    apt-get install logwatch
    
We chose "No Configuration"

    nano /etc/cron.daily/00logwatch

Test your login
*****************


## Setting up minecraft

As root:
********

Get Java

    apt-get install default-jdk
   
   
Install `screen`

    apt-get install screen
    

Create the minecraft folder:

    mkdir /var/minecraft
    chown minecraft:minecraft /var/minecraft
    
    
As minecraft user
_____________________

    cd /var/minecraft
    
    wget -O minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/1.7.4/minecraft_server.1.7.4.jar
    
    screen -S "Minecraft server"
    
    java -Xmx512M -Xms512M -jar minecraft_server.jar nogui


Your Minecraft server is now all set up. 

You can exit out of screen by pressing `Ctrl-a` followed by `d`. To reattach screen, type:

    screen -R

You can change the settings of your server by opening up the server properties file:

     nano /var/minecraft/server.properties
 
Some changes we made:

     nano server.properties

     white-list=true
     allow-nether=false
     enable-command-block=true
     
Change the white-list.txt

     change the whitelist
     
     
## Spigot

Spigot is the new craftbukkit. Get it. In order to get it you'll need to install the `BuildTools.jar` (docs [here](https://www.spigotmc.org/threads/buildtools-updates-information.42865/)) and build it.

    cd /var/minecraft
    wget "https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar" -O BuildTools.jar
    java -jar BuildTools.jar --rev 1.9    

Building takes about 10 minutes.


Make a file called `./craftbukkit.sh`

    #!/bin/sh
    BINDIR=$(dirname "$(readlink -fn "$0")")
    cd "$BINDIR"
    java -Xmx1536M -jar craftbukkit-1.8.8.jar -o true

Change the mode

	chmod +x ./craftbukkit.sh
	
Run it

	./craftbukkit.sh
	
	
## Plugins

### Bending	

Bending instructions are found at [http://dev.bukkit.org/bukkit-plugins/minecraft-last-airbender/files/14-bending-v1-1-0/](http://dev.bukkit.org/bukkit-plugins/minecraft-last-airbender/files/14-bending-v1-1-0/)

    cd /var/minecraft/plugins
    wget http://dev.bukkit.org/media/files/722/749/Bending.jar

### BullionEconomy

[http://dev.bukkit.org/bukkit-plugins/bullioneconomy/](http://dev.bukkit.org/bukkit-plugins/bullioneconomy/))

[http://dev.bukkit.org/bukkit-plugins/bullioneconomy/files/12-bullion-economy-v0-91/](http://dev.bukkit.org/bukkit-plugins/bullioneconomy/files/12-bullion-economy-v0-91/)

    cd /var/minecraft/plugins
    wget http://dev.bukkit.org/media/files/896/296/Bullion-STABLE-BUILD.jar

### Disease (Byte Disease)

[http://dev.bukkit.org/bukkit-plugins/byte-disease/](http://dev.bukkit.org/bukkit-plugins/byte-disease/)

    cd /var/minecraft/plugins
    wget http://dev.bukkit.org/media/files/898/525/Disease-1.7.jar


### MobCatcher Lite

https://www.spigotmc.org/resources/mobcatcher-lite.4065/

Download it to `~/Downloads`

    scp ~/Downloads/MobCatcherLite.jar minecraft@fire:/var/minecraft/plugins
    
### MoveCraft

http://dev.bukkit.org/bukkit-plugins/movecraft/

    wget http://dev.bukkit.org/media/files/882/670/Movecraft.jar

### MythicMobs

http://dev.bukkit.org/bukkit-plugins/mythicmobs/files/57-mythic-mobs-v2-1-6/

    wget http://dev.bukkit.org/media/files/902/616/MythicMobs-2.1.6.jar

### TARDISVortexManipulator

http://dev.bukkit.org/bukkit-plugins/tardisvortexmanipulator/

### CreativeGates

http://dev.bukkit.org/bukkit-plugins/creativegates/

    wget http://dev.bukkit.org/media/files/900/525/CreativeGates.jar


### Factions

http://dev.bukkit.org/bukkit-plugins/factions/

It requires MassiveCore http://dev.bukkit.org/bukkit-plugins/mcore/


    wget http://dev.bukkit.org/media/files/900/523/MassiveCore.jar
    wget http://dev.bukkit.org/bukkit-plugins/factions/files/77-2-8-2/




betonquest (1.8?)
http://dev.bukkit.org/media/files/910/403/BetonQuest.jar

bettercrops
http://dev.bukkit.org/media/files/911/336/better_crops.jar

dynamap
http://dev.bukkit.org/media/files/888/859/dynmap-2.2.jar

multiverse2 (no)
http://ci.onarandombox.com/job/Multiverse-Core/lastSuccessfulBuild/artifact/target/Multiverse-Core-2.5.jar

citizens
http://dev.bukkit.org/media/files/911/551/Citizens.jar

worldedit (1.8?)
http://dev.bukkit.org/media/files/880/435/worldedit-bukkit-6.1.jar

iconomy (very old)
http://dev.bukkit.org/media/files/584/551/iConomy.jar

legendary weapons (1.8)
http://dev.bukkit.org/media/files/878/794/LegendaryWeapons.jar

legendary armor (1.8)
http://dev.bukkit.org/media/files/878/795/LegendaryArmor.jar


enchantplus (1.8)
http://dev.bukkit.org/media/files/890/396/EnchantPlus_1.3.3.jar


tardis (1.8)
http://dev.bukkit.org/media/files/901/423/TARDIS.zip





## Protecting your worlds

Prevent griefing

    gamerule keepInventory true
    gamerule mobGriefing false
 
## Resources

* http://canihostaminecraftserver.com/
* http://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers
* https://www.digitalocean.com/community/articles/how-to-set-up-a-minecraft-server-on-linux
* http://liamodonnell.com/feedingchange/2013/08/two-quick-ways-to-secure-your-minecraft-server/
* http://liamodonnell.com/feedingchange/2013/02/five-essential-plugins-for-minecraft-school-servers/
* http://wiki.bukkit.org/Setting_up_a_server#Linux
* https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04
* https://github.com/Ahtenus/minecraft-init
* https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04
* https://www.digitalocean.com/community/articles/how-to-install-nagios-on-ubuntu-12-10



* http://blog.bitnami.com/2014/03/run-your-minecraft-server-in-cloud-with.html



## Old

## Craftbukkit

<span class="color:red">NOTE: CraftBukkit is no longer available</span>

    wget http://dl.bukkit.org/latest-rb/craftbukkit.jar
    nano craftbukkit.sh

Make the script look like this
    
    #!/bin/sh
    BINDIR=$(dirname "$(readlink -fn "$0")")
    cd "$BINDIR"
    java -Xmx1024M -jar craftbukkit.jar -o true
     
Then
  
    chmod +x craftbukkit.sh
    

To run craftbukkit as your server do:

    ./craftbukkit.sh
    
    
Download the http://dev.bukkit.org/bukkit-plugins/essentials/

    http://dev.bukkit.org/media/files/748/504/Essentials.zip
    
Backup

    http://bukkitbackup.com/
    wget http://www.bukkitbackup.com/download/latest/Backup.jar
 
Edit the config.yml and strings.yml

## Technic Platform (Tekkit, Attack of the B-team)

First you need unzip:

    sudo apt-get install unzip
    
Download the server. For us we want an Attack of the B Team server. So we went to the Technic Platform and searched for the modpack. [Go there](http://www.technicpack.net/modpack/attack-of-the-bteam.552556). Once there you can download the server. You'll need to secure copy this to your server:

    scp ~/Downloads/BTeam_Server_v1.0.12c.zip minecraft@fire:/var/minecraft_attack/

You might be able to do this directly. But this is how we did it.

Unzip that:

    unzip BTeam_Server_v1.0.12c.zip
    
Check that you don't need to change any `server.properties`    

    nano server.properties
    
Copy over the whitelist:

    cp ../minecraft/whitelist.json .

Edit the launcher to max out at 2GB.
    
    nano launch.sh

Make the launcher executable:    
    
    chmod +x launch.sh

Launch:
    
    ./launch.sh

Setup some rules (see above)

    /gamerule keepInventory true
    /gamerule mobGriefing false


### Revenge of the C Team

https://www.atlauncher.com/pack/RevengeoftheCTeam

https://servers.atlauncher.com/server/RevengeOfTheCTeam

# Others

betonquest
bettercrops
dynamap
multiverse2
citizens
worldedit
iconomy
legendary weapons
legendary armor
enchantplus
