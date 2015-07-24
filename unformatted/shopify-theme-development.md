# Shopify Theme Development 

Work on Shopify themes from a local copy with version control (and with panic modes if multiple developers are working).

## Setup

Create a folder:


    mkdir ~/Projects/myshopname
    cd ~/Projects/myshopname
    
Then make that into a `.git` project:

    git init

Next you need a Gemfile so that you can use the custom `shopify_theme` gem (not the public one):


    source 'https://rubygems.org'

    gem 'shopify_theme', git: "git://github.com/jeffrafter/shopify_theme.git"

Now install:

    bundle install
    
Add a `.gitignore`:

    config.yml
    sync.json
    manifest.json


Great, you are ready to go. Next you need to make a `config.yml` for your configuration

    defaults: &defaults
      :api_key: YOUR_PRIVATE_APP_KEY
      :password: YOUR_PRIVATE_APP_PASSWORD
      :store: YOUR_SHOP_NAME.myshopify.com
      :whitelist_files:
      - assets/
      - config/
      - layouts/
      - snippets/
      - templates/
      :ignore_files:
      - .git
      - Gemfile
      - Gemfile.lock
      - README.md
      - sync.json
      - manifest.json
      - config.yml.example

    master:
      <<: *defaults
      :theme_id: YOUR_THEME_ID


In order for the theme tool to work you need a private app. Go to your Shopify Admin and:

* Click on the `Apps` tab on the left
* Click the `Private Apps` button on the upper right
* Click `Create a Private App`
* Give it a title like "Theme"
* Once you save it you should see your `API Key` and `Password`, put those into your config

To get your theme id, click on `Online Store` on the left nav, then `Themes`. If you click `Customize Theme` the theme will open up and the `id` will be the number in the URL. Stick that in your config. 

Once you are all set, you should commit:

    git add .
    git commit -a -m "Initial setup of my theme"
    
## Importing    
    
Then you can grab the latest:

    bundle exec theme import
    
It will download every file locally and create a manifest.

## Making changes

Make a change on shopify and see it from the console (this checks for remote changes):

    bundle exec theme changes

Get that change locally without re-downloading everything:

    bundle exec theme import

Ready to push your local changes up to the remote? Do an export with `dry-run`:

    bundle exec theme export --dry-run

Really ready?

    bundle exec theme export

Are you trying to export but someone made remote changes? Running export would overwrite the changes and we won't let you do that! Stash your local changes, then use `import`. Then re-apply your stash and you can see the diff.

Want to do it live?

    bundle exec theme watch

This will remove any remote files you delete locally (unless you add `--keep-files`) and will upload new files and changed files. If there are remote changes the `watch` command will catch the conflict and not overwrite.

If you need to force overwrite everything you can use the `download` command or clear out your local and `import` again.

    bundle exec theme watch
    
Now as you make changes (add files, remove files, change files) the changes will be synced to your theme up on Shopify (usually in less than a second).

If someone makes a remote change or conflicts the system will stop and let you figure out what the problem is.
    