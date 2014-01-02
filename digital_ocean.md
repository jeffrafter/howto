# Digital Ocean

1. Setup your digital ocean account
2. Add your ssh key
 
Next get the gem
   
    gem install knife-digital_ocean
    
Change your knife.rb:

    cookbook_path    ["cookbooks", "site-cookbooks"]
    node_path        "nodes"
    role_path        "roles"
    environment_path "environments"
    data_bag_path    "data_bags"

    encrypted_data_bag_secret ".chef/encrypted_data_bag_secret"

    knife[:secret_file] = ".chef/encrypted_data_bag_secret"
    knife[:berkshelf_path] = "cookbooks"
    knife[:bootstrap_user] = "sample"

    knife[:digital_ocean_client_id] = "#{ENV['DIGITAL_OCEAN_CLIENT_ID']}"
    knife[:digital_ocean_api_key]   = "#{ENV['DIGITAL_OCEAN_API_ID']}"

Create a droplet:

    DIGITAL_OCEAN_CLIENT_ID=XXXX DIGITAL_OCEAN_API_ID=YYYY knife \
      digital_ocean droplet create -N <FQDN> \
                                   -I <IMAGE ID> \
                                   -L <REGION ID> \
                                   -S <SIZE ID> \
                                   -K <SSH KEY-ID(s), comma-separated> \
                                   -B \
                                   -r "<RUNLIST>"
 
  
  
  digital_ocean_creds = Chef::EncryptedDataBagItem.load("passwords", "digital_ocean")
knife[:digital_ocean_client_id] = "#{ENV['DIGITAL_OCEAN_CLIENT_ID'] || digital_ocean_creds['digital_ocean_client_id']}"
knife[:digital_ocean_api_key]   = "#{ENV['DIGITAL_OCEAN_API_ID'] || digital_ocean_creds['digital_ocean_api_id']}"