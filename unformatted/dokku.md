# Dokku

[Dokku Docs](http://progrium.viewdocs.io/dokku/index)

## On Digital Ocean

Create a new droplet on [Digital Ocean](https://www.digitalocean.com/?refcode=21447a91b37b) (10$ per month). You might opt for a larger plan but this will be enough to run Rails and Sidekiq. When you create the droplet you must give it a `hostname`. This can match your primary domain, or if this is a service for you main domain you can use a subdomain such as `admin.example.com`.

Generally I choose the following options:

* **Select Size**: 10$ per month
* **Select Region**: New York
* **Select Image** (Distribution): Ubuntu 16.04 x64
* **Select Image** (Applications): Dokku 0.4.2
* **Available Settings**: Backups
* **Add SSH Keys**: *Choose one of the SSH keys on your account*

> Note: Do not select the IPv6 as docker doesn't support it well.

Once you create the droplet (about 20 seconds) Digital Ocean immediately begins recording charges for your account. After the droplet has been created, the page will refresh and you can copy the IP address. You can use this address directly or you can setup DNS or just an an entry to your local `/etc/hosts`.

### Complete the dokku install

Open the IP address in your browser.

Change the hostname to match the name of your droplet (i.e., `example.com` or `admin.example.com`). Then click `Finish Setup`.

### First Five Minutes

By default the droplet is not very secure. There are numerous tutorials on how to setup an Ubuntu box in the first five minutes but I generally use a simple script. Login to the server from your local machine via SSH:

    ssh root@example.com

You'll be asked to trust the server; type `yes`. 

On the server:

    git clone git://github.com/jeffrafter/server.git
    cd server
    cp scripts/env.sh.example scripts/env.sh
    nano scripts/env.sh
   
Your settings should look like:

    ROOT_PASSWORD="SOME_PASSWORD"
    DEPLOY_PASSWORD="SOME_PASSWORD"
    ROOT_EMAIL="youremail@gmail.com"
    OPS_EMAIL="youremail@gmail.com"
    VIRTUAL_EMAIL="youremail@gmail.com"
    DOMAIN="$HOSTNAME"
    APPLICATION_NAME="$HOSTNAME"
    
Run the script:

    scripts/setup.sh
    
> Note: once you run this script your SSH will be reconfigured. You will no longer be able to SSH to the server as `root`, but instead must SSH using the `deploy` user. Additionally, the SSH signature of the server will be updated and will not match the `known_hosts` entry from your current login. When you next login (if you don't remove the `~/.ssh/known_hosts` entry) you will receive an error. 

Now that the server is secure you'll need to enable a couple of Dokku specific settings. First make the `dokku` user part of the `ssh-user` group:

    usermod -a -G ssh-user dokku
    
### Create the app


    dokku apps:create yourappname

### Plugins
  
Next you'll want to install any plugins you need (http://dokku.viewdocs.io/dokku/community/plugins/). You'll want to install these plugins on the server as root: This is my standard set:
  
    sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git
    dokku postgres:create yourappname-database
    dokku postgres:link yourappname-database yourappname
    
    sudo dokku plugin:install https://github.com/dokku/dokku-redis.git redis
    dokku redis:create yourappname-redis
    dokku redis:link yourappname-database yourappname

> Note if you closed your previous SSH session you can reconnect as the `deploy` user and then switch the `root` user using `sudo` or prefix these commands with `sudo`.
    
## Setting up your local machine SSH

There are some cases where you want to Request a TTY when you run `dokku` commands (this is generally the case, though it may interrupt some functionality like importing a database dump). To do this you need to modify your local `~/.ssh/config`. Add the following:

    Host yourserver.com
    RequestTTY yes

## Making sure you have the dokku client

To install the bash client simply clone it into your home folder:

    git clone git://github.com/progrium/dokku.git ~/.dokku

In your `~/.bashrc` or `~/.bash_profile` you'll want the following alias:

    alias dokku='$HOME/.dokku/contrib/dokku_client.sh'
    
## Deploying the application 

At this point you are ready to create your application. Within your project folder on your local machine you want to add a remote for `dokku`:

    git remote add dokku dokku@example.com:example.com
    
In this case the hostname (example.com) and the application name (example.com) are the same. Once added, push the application:

    git push dokku master
    
This should deploy the application, though you may need a couple more things before it is operational.        

## Working with the database

From your local machine:
  
    dokku postgres:create example.com
    
And link it:

    dokku postgres:link example.com example.com
    

Migrating your database is not automatic:

    dokku run bundle exec rake db:migrate  

To seed your database:

    dokku run bundle exec rake db:seed
    
## Working with Redis    
    
From your local machine:
    
    dokku redis:create example.com

And link it:

    dokku redis:link example.com example.com
    
## Domains

For some reason my app was deployed and there was no `VHOST` and therefore the default `nginx` configuration wasn't generated. To solve the problem, first ensure that the `NO_VHOST` setting is not set:

    dokku config
    
If it is set (`1`) then unset it:

    dokku config:unset NO_VHOST

Then add the domain:

    dokku domains:add example.com example.com

## Config

I set my config up in one command from my local:

    dokku config:set `cat .env | grep -v RACK_ENV | grep -v PORT`
    


================


### Firewalls, iptables and docker

> Note this did not seem to be needed as of 0.4.2?

Currently the state of the art is confusing. [This is the simplest tutorial](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-networking-and-communication). Docker has multiple networking modes but by default will allow everything to be open and accessible from the outside world.

Within docker networking there are generally four possible goals (with some overlap):

1. Allow docker containers to speak to all other docker containers
2. Allow docker containers to speak to specific docker containers
3. Allow docker containers to speak to the outside world
4. Allow docker containers to listen to the outside world

Given that docker is generally running on a host (i.e., Ubuntu) it is possible that you also want to:

1. Protect the host with a blanket DENY to prevent malicious connections
2. Set rules and bans when someone abuses connecting through to a container

Let's start with the basics. Imagine docker running on Ubuntu 14.04 with a `ufw` (Ubuntu's Uncomplicated Firewall) setup as follows:

    ufw default deny incoming
    ufw allow 22
    ufw allow 443
    ufw allow 80
    ufw enable
    
This will block all external incoming communications except SSH, HTTP and HTTPS. By default this applies to all interfaces including the `docker0` interface meaning that no inter-container communications will work. If you try to view your app in a browser you will probably see an `nginx` error:

    504 Gateway Timeout

The following rule allows you accept all inter-container communication coming in on the `docker0` interface and going back to the destination IP of your `docker0` inet (this address can change per host, to find yours `ifconfig docker0` and see the `inet addr`)

    # The answer:
    ufw allow in on docker0 to 172.17.42.1
    ufw reload

Without even running a `dokku ps:restart` your app should become available.

Apart from testing the website itself, it is useful to test one of the services. For example, if you are running redis, then by default, without the firewall enabled you should be able to telnet to your host on the mapped port (the port 6379 will not be published by default, it will be mapped, you can see the mapping by running `docker ps`)

     telnet yourhost.com 44444 # where 44444 is the mapped port
     
If it connects then Redis is exposed to the world. After enabling the firewall it should no longer be accessible. Once you add the `allow` rule for `docker0` you should verify that it is still protected (which it will be).     

#### Footnotes:

* [This answer](http://serverfault.com/questions/647822/security-implications-of-setting-ufw-default-forward-policy-to-accept) has a lot of really great insight into the problems.  
* [Docker networking](https://docs.docker.com/articles/networking/)
* [Connecting via docker links](https://docs.docker.com/userguide/dockerlinks/)
* [Stop the firewall blocking](http://stackoverflow.com/questions/17394241/my-firewall-is-blocking-network-connections-from-the-docker-container-to-outside)






    
## Seeing logs from rails

If you have centralized logging from your local machine you should be able to run:

    dokku logs -t

Otherwise:

1. Make sure your `production.rb` has a log_level set to `debug`.
2. SSH to the machine
3. Run `docker ps`
4. Find the container id of the container that is running `web` and `docker attach <container_id>`
5. Watch the logs happen (these are the default logs as they pipe to STDOUT on the rails process)
6. To quit you need to type `^[` which on OSX is ALT+Enter.
  



# Swap

If you are seeing this error:

    runtime: panic before malloc heap initialized
    fatal error: runtime: cannot allocate heap metadata

You are running out of disk space. This is [fairly common](https://github.com/docker/docker/issues/1555) on smaller cloud instances, especially when deploying as the number of containers is doubled. To get around this you can add Swap for your memory, which will slow down your server, but keep it running.

    sudo dd if=/dev/zero of=/swapfile bs=1MB count=4096
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    
Next you need to add this to `/etc/fstab`:

    /swapfile   none    swap    sw    0   0
    
You should tweak the swappiness lower and tweak the inode caching. In `/etc/sysctl.conf`:

    vm.swappiness=10
    vm.vfs_cache_pressure=50

#### Footnotes

* [https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04)

## SSL

Get your [SSL](../ssl.md) certificate.

Then:

    cp YOURSITE.com.key server.key
    cp YOURSITE.com.crt server.crt
    tar cf certs.tar server.key server.crt
    
Then from your dokku folder:    

    dokku nginx:import-ssl YOURAPP < certs.tar
    
Then you'll need to push your application to rebuild.

You'll also possibly want to disable VHOSTS

    dokku config:set "NO_VHOST=1"


  
# TODO
  
? Securing the droplet (see server security)
? SSL (there are dokku specific ways to do this, but also see Cloudflare)
? Surviving a docker process crash
? Surviving a reboot

## Footnotes

* http://dev.housetrip.com/2014/07/06/deploy-rails-and-postgresql-app-to-dokku/
* http://donpottinger.net/blog/2014/11/17/bye-bye-heroku-hello-dokku.html
* http://donpottinger.net/blog/2014/11/22/bye-bye-heroku-hello-dokku-part-2.html
* http://donpottinger.net/blog/2014/11/25/postgres-backups-with-dokku.html

## Scratch

Some attempts at getting around the ufw problem:  
      
    # First try
    ufw allow in on docker0 from 127.0.0.1
    ufw allow out on docker0 to 127.0.0.1
    ufw reload

    # Second try
    ufw allow in on docker0 from 0.0.0.0
    ufw allow out on docker0 to 0.0.0.0
    ufw reload
          
    # Third try, the IP address for docker0: 172.17.42.1
    ufw allow in on docker0 from 172.17.42.1
    ufw allow out on docker0 to 172.17.42.1
    ufw reload

## Adding another SSH Key

Upload a public key to the server, then switch to root:

    # cat PUBKEYFILE.pub | sshcommand acl-add dokku
