# Advanced Bootstrapping

When using chef and chef-solo to provision your server you need a couple of tools on the server: primarily ruby and chef. With these installed you can use chef recipes to install everything else. 

The knife solo command allows you to "prepare" a server:

    knife solo prepare vagrant@dev.sample.com -p 2222 -VV --identity-file ~/.vagrant.d/insecure_private_key

This will install a version of ruby and chef and output node information. In the [Provisioning](provisioning.md) how-to we pre-defined the node information and don't use the output anyway. 

Knife-solo allows you to define your own bootstrap template. To do this, create a new file in your `chef` folder called `bootstrap/ubuntu-12.04-lts.erb`

    bash -c '
    <%= "export http_proxy=\"#{knife_config[:bootstrap_proxy]}\"" if knife_config[:bootstrap_proxy] -%>

    # THIS SCRIPT MUST BE RUN AS ROOT

    # Install chef (and ruby)
    curl -O -L http://www.opscode.com/chef/install.sh
    sh install.sh

    BOOTSTRAP_USER=<%= knife_config[:bootstrap_user] || ENV['BOOTSTRAP_USER'] %>
    BOOTSTRAP_GROUP=<%= knife_config[:bootstrap_group] || ENV['BOOTSTRAP_GROUP'] || '$BOOTSTRAP_USER' %>
    BOOTSTRAP_KEY=<%= knife_config[:bootstrap_key] || ENV['BOOTSTRAP_KEY'] || `cat ~/.ssh/id_rsa.pub`.chomp %>

    # Add admin group
    (cat /etc/group | grep -E '\b$BOOTSTRAP_GROUP\b') || sudo groupadd $BOOTSTRAP_GROUP

    # Add admin user
    (cat /etc/passwd | grep -E "\b$BOOTSTRAP_USER\b:x") || useradd -m -s /bin/bash $BOOTSTRAP_USER -g $BOOTSTRAP_GROUP

    # sudoless access for admin user
    (cat /etc/sudoers | grep -E "^$BOOTSTRAP_USER\b.*NOPASSWD") || echo "$BOOTSTRAP_USER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

    # Configure SSH
    SSH_DIR=/home/$BOOTSTRAP_USER/.ssh
    mkdir -p -m 700 $SSH_DIR
    echo $SSH_KEY > $SSH_DIR/authorized_keys
    chmod 600 $SSH_DIR/authorized_keys
    chown -R $BOOTSTRAP_USER:$BOOTSTRAP_GROUP $SSH_DIR

    # Disable password access
    sed -E -i "s/.*PasswordAuthentication.*/PasswordAuthentication no/" /etc/ssh/sshd_config
    sed -E -i "s/.*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication no/" /etc/ssh/sshd_config
    restart ssh

    apt-get update

    # Specifying noninteractive mode gets past the grub-rc error on vagrant
    DEBIAN_FRONTEND=noninteractive apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" dist-upgrade
    DEBIAN_FRONTEND=noninteractive apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" upgrade --force-yes

    # Set firewall rules
    ufw default deny
    ufw allow ssh
    ufw allow 80/tcp
    ufw allow 443/tcp
    echo y | ufw enable

    reboot now
    '

You will also need to add an additional configuration item in your `.chef/knife.rb`:

    knife[:bootstrap_user] = "sample"
    
This user and group will be added by default and your public key will be added to the SSH authorized keys. You can override the group by specifying the `bootstrap_group` and you can override the key by specifying the `bootstrap_key`. 


    knife solo prepare vagrant@dev.sample.com -p 2222 -VV --identity-file ~/.vagrant.d/insecure_private_key --template-file bootstrap/ubuntu-12.04-lts.erb