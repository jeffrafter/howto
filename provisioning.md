# Provisioning

When you want to reliably deploy server infrastructure (dev-ops) you have a variety of options. You could do it manually, typing in all of the commands. You could have a script that initializes everything (there may be a particular setup for your cloud provider, e.g., StackScripts). You could use a provisioning toolkit like Chef, Puppet, or Ansible. 

In some respects, all of these are out of date. Modern dev-ops relies on pre-built images (using AMIs, Droplets or the like directly) or by using something like Packer. Tools like Synapse act as command and control to provision images.

For now, we'll use Chef. It is a pretty reliable system albeit with a few quirks.

## Chef, Knife and Berkshelf

Chef describes it's provisioning configuration as "recipes" and "cookbooks". Most people that use chef use it with the proper Chef/Opscode hosted platform. Using the Opscode platform you can quickly centralize your cookbooks and provision your servers with those recipes and the knife command. 

Instead of using the hosted platform, I prefer to put my cookbooks and recipes into a git repository and provision from there. You might argue that I am trading one dependency for another and you would probably be right. Luckily Chef has a "solo" mode that you can use with "knife-solo" to provision without the hosted platform.

You can choose to keep your cookbooks in a separate repository from your application, and in most situations this is recommended. However, if you are just starting your application and have a small (or no) team then keeping everything together is just fine.

### Getting started

You need to have some local gems installed:

    gem install knife-solo
    gem install knife-solo_data_bag
    gem install berkshelf

You could put these into your `Gemfile` but then you would need to bundle exec the commands. For simplicity I have just installed them.

From your Rails root folder you want to create a folder for your cookbooks:

    knife solo init chef
    
You should see:

    WARNING: No knife configuration file found
    Creating kitchen...
    Creating knife.rb in kitchen...
    Creating cupboards...
    Setting up Berkshelf...
    
This will create the basic structure you need in a folder called `chef`. The structure should look like:

    chef/    
    ├── .chef
    │   └── knife.rb
    ├── .gitignore
    ├── Berksfile    
    ├── cookbooks
    ├── data_bags
    ├── environments
    ├── nodes
    ├── roles
    └── site-cookbooks

To break this down:

* `.chef` - this is a hidden folder that contains configuration information for knife. Eventually we will keep information about encrypted data bags here as well, but other than that you don't need to worry about this folder.

* `.gitignore` - this is a folder specific ignore. Currently it is set to only ignore the `cookbooks` folder which is all we need to worry about.

* `Berksfile` - this is like a `Gemfile` but instead of managing gems your application relies on, it manages cookbooks your server relies on. As we make changes to `Berksfile` and use knife, it will download and cache cookbook dependencies in the `cookbooks` folder (and in the `~/.berkshelf` folder) and keep track of those in `Berksfile.lock`.

* `cookbooks` - a folder to store cookbook dependencies. You should not make any manual changes in this folder as they will be overwritten. 

* `data_bags` - a folder to store configuration (and encrypted configuration) files in JSON format.

* `environments` - a folder to store environment specific configuration.

* `nodes` - a folder to store node specific configuration. The nodes names correspond to provisioning destinations when using the knife command.

* `roles` - a folder for storing role specific information including the name of the node and the run list.

* `site-cookbooks` - a folder to store your site-specific cookbooks. This is where you should make changes that won't be overwritten by Berkshelf.

### Berksfile

By default, I setup a server to use Ruby (via rbenv), Postgres, Redis, and Nginx. You also need some basics which are included here:

    # You need to indicate where Berkshelf should get community cookbooks
    site :opscode

    # When using knife solo you need chef-solo-search
    cookbook 'chef-solo-search', git: "https://github.com/edelight/chef-solo-search.git"

    # Basic server information
    cookbook 'apt'
    cookbook 'ntp'
    cookbook 'cron'
    cookbook 'sudo'
    cookbook 'users'
    cookbook 'monit'
    cookbook 'vim'
    cookbook 'imagemagick'

    # Ruby management
    cookbook 'ruby_build', git: 'https://github.com/fnichol/chef-ruby_build'
    cookbook 'rbenv', git: 'https://github.com/fnichol/chef-rbenv'

    # We need node for building assets
    cookbook 'node'
    
    # Nginx doesn't come with support for the upload module by default
    cookbook 'nginx'
    cookbook 'nginx_upload_module', git: "https://github.com/jeffrafter/nginx_upload_module.git"
    cookbook 'unicorn'

    # Data storage repos
    cookbook 'redis', git: 'https://github.com/miah/chef-redis'
    cookbook 'postgresql'
    cookbook 'database'

Copy this into your `Berksfile` and then (from within your `chef` folder) run:

    `berks install`
    
### Data bags

At this point it best to get some of the basic configuration setup. Most of this is managed via JSON in files called "data bags". Any information that will be used across all of your server instances can be setup inside of a file called `data_bags/default/YOURAPP.json`. In this case `data_bags/default/sample.json`. Really, you can call the file whatever you like, this is just convention.

    {
      "app": {
        "name": "sample"
      },
      "authorization": {
        "sudo": {
          "passwordless": true
        }
      },
      "sample": {
        "deploy_to": "/var/www/sample",
        "user": "sample",
        "group": "sample",
        "database": {
          "name": "sample",
          "pool": 50
        }
      },
      "ruby_build": {
        "git_url": "git://github.com/sstephenson/ruby-build.git",
        "git_ref": "v20131211",
        "upgrade": "sync"
      },
      "rbenv": {
        "user_installs": [
          {
            "user" : "sample",
            "rubies" : ["2.0.0-p353"],
            "global" : "2.0.0-p353",
            "gems" : {
              "2.0.0-p353" : [
                { "name" : "bundler" },
                { "name" : "backup" }
              ]
            }
          }
        ]
      },
      "nginx": {
        "source": {
          "modules": [
            "nginx_upload_module::upload_module",
            "nginx::http_gzip_static_module",
            "nginx::http_ssl_module",
            "nginx::upload_progress_module"
          ]
        }
      },
      "monit": {
        "notify_email": "YOUREMAIL@YOURDOMAIN.com"
      },
      "build_essential": {
        "compiletime": true
      },
      "cron": {
        "tasks": [
        ]
      },
      "run_list": [
      ]
    }

This covers most of the basics. Node specific information will be stored in a node JSON file (for example, environment, cron tasks, run lists). The one caveat is `build_essential`: the `compiletime` setting is required to make chef's dependency resolver do the right thing in the right order.

We'll also need a data_bag to store information about users (for example the user that will deploy the application and which users can ssh into the server). In `data_bags/users/YOURAPPNAME.json` add the following:

    {
      "id": "sample",
      "groups": ["sysadmin"],
      "shell": "/bin/bash",
      "ssh_keys": [
        "YOUR SSH PUBLIC KEY",
      ]
    }

Here I create a user with the same name as my app. This is my deploy user. Most people actually call this user "deploy" but because I tend to work on multiple sites simultaneously it is sometimes helpful for me to remember where I am by the name. Your SSH public key is safe to include here, you can get it by running:

    cat ~/.ssh/id_rsa.pub
    
**Do not include your private key** (the one without .pub). 

Note: you can also include a `password` option here with a hashed password. According to the help you can generate a hashed password:

    openssl passwd -1 "plaintextpassword"
    
However, this might not be safe as it creates an MD5 hash which is easily guessable. Instead use (if you are targeting Ubuntu):

    mkpasswd -m sha-512
    
Or better yet, don't include a password. You don't need one.    

### Encrypted Data Bags

Some information should not be stored in your repository. Other information can be stored in your repository but should be encrypted. To create an encrypted data bag you need to first make sure you have a secret file (in your `chef` folder):

    openssl rand -base64 512 > .chef/encrypted_data_bag_secret
        
**Make sure you ignore this file in your .gitignore**. In the default `.gitignore` in this how to, we already ignore this file. However, you might want to add it to the `.gitignore` inside of the `chef` folder.

You need to tell knife that this is your secret in your .chef/knife.rb file. It is possible to override that setting on the command line, but specifying it in the config is simpler. Inside your `chef/knife.rb`:

    cookbook_path ["cookbooks", "site-cookbooks"]
    node_path     "nodes"
    role_path     "roles"
    data_bag_path "data_bags"
    encrypted_data_bag_secret ".chef/encrypted_data_bag_secret" 

    knife[:berkshelf_path] = "cookbooks"


Once you have that you should be able to create a new encrypted datbag:

    knife solo data bag create passwords aws 
    
This creates a new databag folder called "passwords" and creates a new databag inside of it called aws.json. The command will immediately open vim for you to create your settings:

    1 {
    2   "id": "aws",
    3   "aws_access_key_id": "YOUR_ACCESS_KEY_ID",
    4   "aws_secret_access_key": "YOUR_SECRET_ACCESS_KEY"
    5 }

Once you write and quit the file will be encrypted and can be safely added to your repository. If you need to change the settings you can use the edit command:

    knife solo data bag edit passwords aws 
    
You can use this same pattern for your database users:

    knife solo data bag create passwords postgresql
    
Create two user entries:

    1 {
    2   "id": "postgresql",
    3   "postgres": "A COMPLEX PASSWORD",
    4   "sample": "ANOTHER COMPLEX PASSWORD"
    5 }

Later, when installing Postgres and setting up the `database.yml` for our Rails application on the server, we'll use the decrypted form of these passwords. Here the `postgres` is the Postgres user and `sample` is the name of the database user specified for your Rails environment (you might choose `deploy` here if you did above).

Now you can safely store the sensitive information in your repository and use the decrypted values during the run of your cookbook. Note however that the values will be decrypted in memory when they are used. 

### Roles

Roles are simple groupings of cookbooks that run on specific kinds of servers (for example, app servers, worker servers, web servers and database servers). For any server we will need to setup the users and secure the box. We'll also want to provide some helpful tools and monitoring. Because this is shared, it is best to create a base role in `roles/base.rb`:

    name 'base'

    run_list(
      'recipe[vim]',
      'recipe[chef-solo-search]',
      'recipe[ntp]',
      'recipe[cron]',
      'recipe[sudo]',
      'recipe[users]',
      'recipe[users::sysadmins]',
      'recipe[monit]',
      'recipe[monit::ubuntu12fix]'
    )

This just lists a set of recipes to be run. We'll refer to this role later. 

For the web role add the following to `roles/web.rb`:

    name 'web'

    run_list(
      "recipe[nginx_upload_module::upload_module]",
      "recipe[nginx::source]"
    )

For the data role add the following to `roles/data.rb`:

    name 'data'

    run_list(
      "recipe[redis::server_package]",
      "recipe[postgresql::server]",
      "recipe[database::postgresql]"
    )

For the app role add the following to `roles/app.rb`:

    name 'app'

    run_list(
      "recipe[ruby_build]",
      "recipe[rbenv::user]",
      "recipe[node]",
      "recipe[imagemagick]"
    )

These roles are somewhat arbitrary when starting out. You might split Redis into a separate role called "cache" for instance. You might create a worker role that includes `imagemagick` and remove it from the app role.

### Nodes

Nodes represent actual servers. When working with a small number of nodes (1-3) this is fairly simple to manage. As the complexity of the system increases and you have multiple app nodes, multiple worker nodes, and need to rely on coordination and service discovery this pattern will probably fall apart. At that point you might want something like Synapse. For now we will start simple.

The first node you should create is a local node for your vagrant box. The name of the node must correspond to the address of the node and you can do this in a couple of ways:

1. You could name the node 127.0.0.1.json knowing you will provision to your local.
2. You could create an entry in your `/etc/hosts` file to give a resolvable domain and name your node correspondingly.
3. You could create an actual DNS entry for your domain which points to 127.0.0.1 and name your node correspondingly.

There are a couple of additional options here which involve DNS, resolvers, etc. But we'll go the easy route and choose option 2. To edit your `/etc/hosts` file you'll need to use `sudo`. This is generally considered bad form as you never want to use `sudo`:

    sudo vim /etc/hosts
    
Add the following entry:

    127.0.0.1 dev.YOURDOMAIN.com
    
You could, in theory bind to 0.0.0.0 which would bind to all interfaces. But this keeps things a little more specific to loopbacks.

Now, to create the node (in your `chef` folder) add the file `nodes/dev.YOURDOMAIN.com.json`:

    {
      "sample": {
        "environment": "production"
      },
      "authorization": {
        "sudo": {
          "users": ["vagrant"],
          "passwordless": true
        }
      },
      "run_list": [
        "role[base]",
        "role[web]",
        "role[data]",
        "role[app]",
        "recipe[sample]"
      ]
    }

First, we set the environment to production. You might want to choose development or staging here. This generally controls which Rails environment is used (and therefore which entries in the config *.yml files are used).

We have also added `sudo` authorization for the `vagrant` user which will exist by default on your Vagrant instance. This is really helpful but not absolutely required (the vagrant user has sudo privileges by default, but the other cookbooks will remove it without this setting).

Finally we specify the `run_list` which is just a list of all our roles and one recipe we haven't seen yet. By including all of the roles we are saying that this node (our local vagrant) will act like the web server, database server, and app server all at once. You could also create multiple vagrant boxes and test out mutli-server setups. 

The sample recipe at the end of the list refers to the site-cookbook we will make next. You should name this after your application (here we used "sample").

### Site Cookbook

Cookbooks have a lot of options and can provide a lot of functionality. The Site-cookbook however is pretty specific and you can keep things simple. Once completed your site-cookbook will have the following directory structure:


    chef/    
    └─ site-cookbooks
       └─ sample
          ├─ metadata.rb          
          ├─ attributes
          │  └─ default.rb          
          ├─ recipes
          │  └─ default.rb
          └─ templates
             └─ default
                ├── backup.rb.erb    
                ├── backup-config.rb.erb    
                ├── database.yml.erb    
                ├── inputrc    
                └── monit-redis.conf.erb    
    
The `sample` folder above is the name of the recipe you referred to in your run_list and should be named after your application.

The metadata for the cookbook is simple. In `site-cookbooks/sample/metadata.rb` include the following:

    name             'sample'
    version          '0.0.1'

Cookbook attributes include a set of default attributes and the merged in data bags. Create the `site-cookbooks/sample/attributes/default.rb`:

    aws_creds = Chef::EncryptedDataBagItem.load("passwords", "aws")
    default['sample']['aws'] = {
      :aws_access_key_id => aws_creds['aws_access_key_id'],
      :aws_secret_access_key => aws_creds['aws_secret_access_key']
    }

    pg_creds = Chef::EncryptedDataBagItem.load("passwords", "postgresql")
    default['postgresql']['password']['postgres'] = pg_creds['postgres']

    default['sample']['database'] = {
      :adapter => 'postgresql',
      :name => 'sample',
      :user => 'sample',
      :password => pg_creds['sample'],
      :host => 'localhost'
    }

    default['postgresql']['pg_hba'] = [
      { :type => 'local', :db => 'all', :user => 'postgres', :addr => nil, :method => 'ident' },
      { :type => 'host', :db => 'all', :user => 'all', :addr => '127.0.0.1/32', :method => 'md5' },
      { :type => 'host', :db => 'all', :user => 'all', :addr => '::1/128', :method => 'md5' },
      { :type => 'host', :db => node['sample']['database']['name'], :user => node['sample']['database']['user'], :addr => '127.0.0.1/32', :method => 'md5' },
      { :type => 'local', :db => node['sample']['database']['name'], :user => node['sample']['database']['user'], :addr => nil, :method => 'trust' },
      { :type => 'local', :db => 'all', :user => 'all', :addr => nil, :method => 'ident' }
    ]

The first step in the recipe is to decrypt the encrypted data bags we created earlier. Once decrypted, we assign the values to "default" attributes. Note, again, we are using the name "sample" which corresponds to the application name and the attributes we setup earlier.

Next we setup some defaults for the Postgres configuration (using some of the encrypted values).

With the attributes setup, we can create the default recipe. I tend to add everything I need into the default recipe. You could split the recipe into separate files (for example, shell.rb, database.rb, application.rb, etc.) and then include those with `include_recipe` but the organization is already somewhat Byzantine so I keep it all together. Add the following to the file `site-cookbooks/sample/recipes/default.rb`:

    ## Default attributes
    #
    default_data_bag = data_bag_item("default", "sample")
    node.default_attrs = Chef::Mixin::DeepMerge.merge(node.default_attrs, default_data_bag.to_hash)

    ## Shell
    #
    template ".inputrc" do
      path "/home/#{node['sample']['user']}/.inputrc"
      source "inputrc"
      action :create_if_missing
    end

    ## Database
    #
    postgresql_connection_info = {
      :host => node['sample']['database']['host'],
      :port => node['postgresql']['config']['port'],
      :username => 'postgres',
      :password => node['postgresql']['password']['postgres']
    }

    postgresql_database node['sample']['database']['name'] do
      connection postgresql_connection_info
      action :create
    end

    postgresql_database_user 'sample' do
      connection postgresql_connection_info
      password node['sample']['database']['password']
      database_name node['sample']['database']['name']
      privileges [:all]
      action [:create, :grant]
    end

    ## Application
    #
    shared_dir = "#{node['sample']['deploy_to']}/shared"

    directory shared_dir do
      action :create
      owner node['sample']['user']
      group node['sample']['group']
      mode '0755'
      recursive true
    end

    template "#{shared_dir}/database.yml" do
      source 'database.yml.erb'
      owner node['sample']['user']
      group node['sample']['group']
      mode '0644'
    end

    bash "fix home permissions" do
      user "root"
      cwd "/home/#{node['sample']['user']}"
      code <<-EOH
      chown -R #{node['sample']['user']}:#{node['sample']['group']} .
      EOH
    end

    bash "fix app permissions" do
      user "root"
      cwd "#{node['sample']['deploy_to']}"
      code <<-EOH
      chown -R #{node['sample']['user']}:#{node['sample']['group']} .
      EOH
    end

    ## Backup
    #
    directory "#{shared_dir}/backups/models" do
      action :create
      owner node['sample']['user']
      group node['sample']['group']
      mode '0755'
      recursive true
    end

    template "config.rb" do
      path "#{shared_dir}/backups/config.rb"
      source "backup-config.rb.erb"
      action :create
    end

    template "backup.rb" do
      path "#{shared_dir}/backups/models/backup.rb"
      source "backup.rb.erb"
      variables({
        backup_db_name: node['sample']['database']['name'],
        backup_db_username: 'sample',
        backup_db_password: node['sample']['database']['password'],
        backup_aws_access_key_id: node['sample']['aws']['aws_access_key_id'],
        backup_aws_secret_access_key: node['sample']['aws']['aws_secret_access_key'],
        backup_aws_region: 'us-west-2',
        backup_aws_bucket: 'sample'
      })
      action :create
    end

    ## Cron
    #
    tasks = node[:cron][:tasks] rescue []
    tasks.each do |task|
      cron task[:name] do
        user "sample"
        path "/home/sample/.rbenv/shims:/home/sample/.rbenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        minute task[:minute] || "0"
        hour task[:hour] if task[:hour]
        day task[:day] if task[:day]
        month task[:month] if task[:month]
        command task[:command]
        action :create
      end
    end

    ## Monitoring
    #
    template "/etc/monit/conf.d/redis.conf" do
      source 'monit-redis.conf.erb'
      owner 'root'
      group 'root'
    end

    bash "restart monit" do
      user "root"
      code <<-EOH
      service monit restart
      EOH
    end

There are seven main sections here:

* __Default attributes__: this loads the information from the default data bag and merges the values into our current node.
* __Shell__: this just sets up a .inputrc file which provides some helpful defaults when you are SSH'd into the machine (which you shouldn't need to do). You could omit this.
* __Database__: this section sets up the database users and connection information. You only need this on the database servers, so splitting this out to a separate recipe might be useful. You could also surround this with a test for the current role.
* __Application__: this sets up the default folders for the application deployment. On some servers you won't need this (i.e., a web server acting only as a proxy or a database server on it's own).
* __Backup__: this setups the configuration for the `backup` gem. Note I am assuming a very specific configuration (backing up postgres to S3). Clearly, this is not needed on every server, but by default the backup is not run. That is delegated to a `cron` task.
* __Cron__: Cron tasks are not included directly in the site-cookbook. Instead cron tasks should be added to specific nodes. This way you don't have multiple machines trying to do the same thing simultaneously.
* __Monitoring__: this sets up a basic monit task for Redis. Much more could be added to this.

To support this recipe you need to add a set of templates to the folder `site-cookbooks/sample/templates/default/`. First create `backup-config.rb.erb`:

    ##
    # Load all models from the models directory.
    Dir[File.join(File.dirname(Config.config_file), "models", "*.rb")].each do |model|
      instance_eval(File.read(model))
    end

Then the `backup.rb.erb` template:

    ## Basic database backup
    Model.new(:backup, 'Backup database') do
      ##
      # Split [Splitter]
      #
      # Split the backup file in to chunks of 4096 megabytes
      # if the backup file size exceeds 4096 megabytes
      #
      split_into_chunks_of <%= @backup_split || 4096 %>

      ##
      # Backup for the Postgres database [Database]
      #
      database PostgreSQL do |db|
        db.name               = "<%= @backup_db_name %>"
        db.username           = "<%= @backup_db_username %>"
        db.password           = "<%= @backup_db_password %>"
      end

      ##
      # Gzip [Compressor]
      #
      compress_with Gzip

      ##
      # Storing the backup to S3 [Storage]
      #
      store_with S3 do |s3|
        s3.access_key_id     = "<%= @backup_aws_access_key_id %>"
        s3.secret_access_key = "<%= @backup_aws_secret_access_key %>"
        s3.bucket            = "<%= @backup_aws_bucket %>"
        s3.region            = "<%= @backup_aws_region || 'us-east-1' %>"
        s3.path              = "<%= @backup_aws_path || '/backups' %>"
        s3.keep              = <%= @backup_aws_keep || 10 %>
      end
    end

You can see more about the backup gem and its configuration [here](https://github.com/meskyanichi/backup).

The cookbook will automatically create a database.yml file for our application using the encrypted values specified earlier. To do this create the template `database.yml.erb`:

    <%= node['sample']['environment'] %>:
      adapter: postgresql
      encoding: unicode
      database: <%= node['sample']['database']['name'] %>
      pool: <%= node['sample']['database']['pool'] || 50 %>
      username: <%= node['sample']['database']['user'] %>
      password: <%= node['sample']['database']['password'] %>
      host: <%= node['sample']['database']['host'] || '127.0.0.1' %>

Note that we default the pool size to 50. This leaves room for double the default set of 25 Sidekiq workers (which is the max of any given process we are concerned with).

Basic monitoring with monit can be configured in the `monit-redis.conf.erb` template:

    check process redis-server
        with pidfile "<%= node['redis']['config']['pidfile'] %>"
        start program = "/etc/init.d/redis-server start"
        stop program = "/etc/init.d/redis-server stop"
        if 2 restarts within 3 cycles then timeout
        if totalmem > 100 Mb then alert
        if children > 255 for 5 cycles then stop
        if cpu usage > 95% for 3 cycles then restart
        if failed host 127.0.0.1 port 6379 then restart
        if 5 restarts within 5 cycles then timeout

The `inputrc` template is really just a set of my preferences:

    set show-all-if-ambiguous on
    set completion-ignore-case on
    set bell-style none

    set prefer-visible-bell

    "\e[A": history-search-backward
    "\e[B": history-search-forward
    "\e[5C": forward-word
    "\e[5D": backward-word
    "\e\e[C": forward-word
    "\e\e[D": backward-word

    $if Bash
      Space: magic-space
    $endif

### Bootstrapping and Provisioning your Vagrant Box

If you followed the steps from [Vagrant](vagrant.md) how-to then you should have a running vagrant box. You might have halted the box:

    # Go to your Rails root
    cd /projects/sample
    vagrant up
    
If you see a message that vagrant is already running that is fine.

Great, you should now have a running vagrant box. Now we need to bootstrap the box with the basics we need to run our chef scripts. 

> Note: At this point you can use the built-in tools to bootstrap the server, or you can define your own template. For an example of this see [Advanced Bootstrapping](advanced_bootrapping.md) instead of using the following command.

Knife allows you to "prepare" the box (which will setup chef and ruby):

    cd chef
    knife solo prepare vagrant@dev.sample.com -p 2222 -VV --identity-file ~/.vagrant.d/insecure_private_key

Notice that we used the `vagrant` user to prepare. By default this user is built in to Vagrant. If you are prompted with a password (for sudo) it should be "vagrant". We also specified that the SSH session will happen on port 2222 which we bound in the `Vagrantfile`.

In the above command we've turned on -VV (two capital letter 'V') for very verbose output, though this is not necessary. Some of the chef commands take a really long time and this helps to see how far along things are. When chef runs initially it is compiling packages and is slow, so be patient. To improve the performance see the notes on Packer and [CheckIntall](checkinstall.md). Once complete you'll see:

    DEBUG: Node config 'nodes/dev.sample.com.json' already exists

Which means your Vagrant box is ready for provisioning. In order to actually run the provisioning you'll need the `encrypted_data_bag_secret` file in the `chef/.chef` folder. You created this above so you have a copy, but this isn't included in the repository for security reasons. So if other members of your team want to provision they will need this file.

Once you have _prepared_ the vagrant box you should be able to _cook_ it:

    knife solo cook vagrant@dev.sample.com -p 2222 --identity-file ~/.vagrant.d/insecure_private_key


Cooking the Vagrant runs all of the chef scripts. Doing this adds a `sample` user to the box and, assuming your public key is added in `chef/data_bags/users/sample.json` you will be able to SSH directly to the box.

    ssh -p 2222 sample@dev.sample.com

Now, you can recook the box as needed (note you have to use `dev.sample.com` to match the name of the node and you'll want to use the newly created `sample` user):

    knife solo cook sample@dev.sample.com -p 2222

This process should be pretty fast.


## TODO

* Look into custom bootstrap script here https://github.com/opscode/chef/blob/master/lib/chef/knife/bootstrap/ubuntu12.04-gems.erb


knife bootstrap --template-file bootstrap/ubuntu-12.04-lts.erb -u vagrant dev.exposely.com echo '{"run_list":[]}' > nodes/dev.exposely.com.json

knife bootstrap --template-file bootstrap/ubuntu-12.04-lts.erb -p 2222 -u vagrant dev.exposely.com  --identity-file ~/.vagrant.d/insecure_private_key


knife bootstrap --template-file bootstrap/ubuntu-12.04-lts.erb -p 2222 -u root dev.exposely.com  -i ~/.vagrant.d/insecure_private_key



## Resources

* [knife-solo](http://matschaffer.github.io/knife-solo/)
* [knife-solo_data_bag](http://thbishop.com/knife-solo_data_bag/)
* [Backup gem](https://github.com/meskyanichi/backup)