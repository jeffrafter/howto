# Advanced Bootstrapping

When using chef and chef-solo to provision your server you need a couple of tools on the server: primarily ruby and chef. With these installed you can use chef recipes to install everything else. 

The knife solo command allows you to "prepare" a server:

    knife solo prepare vagrant@dev.sample.com -p 2222 -VV --identity-file ~/.vagrant.d/insecure_private_key

This will install a version of ruby and chef and output node information. In the [Provisioning](provisioning.md) how-to we pre-defined the node information and don't use the output anyway. 

Knife-solo allows you to define your own bootstrap template. To do this, create a new file in your `chef` folder called `bootstrap/ubuntu-12.04-lts.erb`

