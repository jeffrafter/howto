# Dokku

[Dokku Docs](http://progrium.viewdocs.io/dokku/index)

## On Digital Ocean

1. Create a new droplet on Digital Ccean (10$ per month)
2. Make sure you select your SSH key to be installed
3. Select the Dokku application 
4. Once provisioned open the IP of the new droplet in your browser
5. Setup the host on your local machine or on your DNS provider

    ```
    sudo nano /etc/hosts 
    ```

6. SSH into droplet and add the following plugins

   ```
   ssh root@YOUR_NEW_IP
   ```
	
8. Run this code:

    ```     
    mkdir -p /var/log/dokku # For logging-supervisord
    chown dokku:dokku /var/log/dokku

    cd /var/lib/dokku/plugins
    git clone https://github.com/Kloadut/dokku-pg-plugin postgresql
    git clone https://github.com/luxifer/dokku-redis-plugin redis
    git clone https://github.com/sehrope/dokku-logging-supervisord logging-supervisord
    git clone https://github.com/rlaneve/dokku-link.git link
    dokku plugins-install
    ```
    
Currently the `dokku-logging-supervisord` plugin will hang when deploying after it re-installs `supervisord` (or skips the install). Just hit enter (twice?) to avoid this. 

A branch is underway to incorporate the scaling and Procfile handling of the `dokku-logging-supervisord` directly into dokku. It is unclear if it will still use `supervisord` to manage process crashes. Additionally, it will not include log centralization which may be a priority.
    

## Setting up your ssh

There are some cases where you want to Request a TTY when you run dokku commands:

    Host yourserver.com
    RequestTTY yes

On the server you may have limited SSH access to a specific group. You may need to add the dokku user to that group:

    usermod -a -G ssh-user dokku


## Making sure you have the dokku client

To install the bash client simply clone it into your home folder:

    git clone git@github.com:progrium/dokku.git ~/.dokku

In your `~/.bashrc` you'll want the following alias:

    alias dokku='$HOME/.dokku/contrib/dokku_client.sh'


## Working with the database

From your local machine:
  
    dokku postgresql:create your_app_name

Migrating your database is not automatic:

    dokku run bundle exec rake db:migrate  

To seed your database:

    dokku run bundle exec rake db:seed
    
## Working with Redis    
    
From your local machine:
    
    dokku redis:create your_app_name

    
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
  

## Firewalls, iptables and docker

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

# Troubleshooting

For some reason my app was deployed and there was no VHOST and therefore no nginx. To solve the problem:

     dokku domains:add api.rpl.cat

  
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