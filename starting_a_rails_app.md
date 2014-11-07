Starting a Rails app
====================

This is only the basics. Getting your repository, your organization and getting some of the basic spec infrastructure setup.

## Ruby and Rails

_Get your ruby ready_. I use rbenv and ruby-build. There are a few other options available (rvm, etc.) but this is the one that has worked for me:

    brew upgrade ruby-build
    rbenv install -l

    # Install the newest
    rbenv install 2.1.3
    rbenv global 2.1.3
    gem install rake rails bundler foreman mailcatcher   
    
_Install your basic gems_. If you have just installed a new version of ruby you might need to `gem install bundler rake rails` 

_Initialize the project_. I have a folder called projects which I can access via /projects. This is just a symlink to ~/Projects:

    cd /projects

Within my projects folder I will _start the new Rails app_ and this will create an application folder and generate all of the initial files:

    rails new sample -d postgresql --skip-test-unit
    cd sample

> Note: if you are creating an app and plan to deploy to heroku you probably want to use postgres as your database. If you are using another platform you may not want postgres. In that case you should just create the app without that set: `rails new sample --skip-test-unit`

In this case my application folder is /projects/sample. This is my "Rails Root" path for my application (on my development machine).

You want to use the latest ruby there (this creates a .ruby-version file):

    rbenv local 2.1.2

_Remove the test folder_. you want to use specs instead.

    rm -rf test
    
## Config

Setup the most basic `config/application.rb`. By default you only need the
following two options:

    # Set the test framework to rspec
    config.test_framework = :rspec

    # Set the default encoding
    config.encoding = "utf-8"

For Heroku, you'll need to configure the asset pipeline to compile. In `config/environments/production.rb`:
    
    # Do not fallback to assets pipeline if a precompiled asset is missed.
    config.assets.compile = true

If you are sending email then you can use [Mailcatcher](http://mailcatcher.me) locally. This allows you to catch all email being sent from Rails and view it.

> To install Mailcatcher just `gem install mailcatcher`. Then run `mailcatcher`. When running you can open a browser to [see the mails](http://localhost:1080).
    
You'll need to configure mailcatcher in `config/environments/development.rb`:
      
    # Mailcatcher
    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025, :domain => "localhost:3000" }

You'll also want to set the host for the default urls:

    config.action_mailer.default_url_options = { host: "http://localhost:3000" }

You should set the `default_url_options` for the production and test environments as well.

If you use Safari when you develop you may run into refresh bugs because of Safari's poor handling of 304 Not Modified. If so, add this to your `config/environments/development.rb`:

    # Fix the Safari 304 not modified bug (https://bugs.webkit.org/show_bug.cgi?id=32829)
    config.middleware.delete Rack::ETag

## Gems

Setup the `Gemfile`. This sample Gemfile has quite a few helpful defaults, but you should compare it with the generated Gemfile and pick and choose:

    # You should specify the ruby version for Heroku
    ruby "2.1.2"

    source 'https://rubygems.org'

    # Standard rails, without coffee-script
    gem 'rails', '4.1.6'
    gem 'sass-rails'
    gem 'uglifier'
    gem 'jquery-rails'

    # Processes
    gem 'foreman'
    gem 'pg'
    gem 'sidekiq'
    gem 'thin'

    # Heroku suggests the rails_12factor gem, but this should only
    # be used in production
    group :production do
      gem 'rails_12factor'
    end

    # Kits
    gem 'authkit', git: 'https://github.com/jeffrafter/authkit'
    gem 'errorkit', git: 'https://github.com/jeffrafter/errorkit'
    gem 'notifykit', git: 'https://github.com/jeffrafter/notifykit'
    gem 'uikit', git: 'https://github.com/jeffrafter/uikit'

    # For authkit
    gem "active_model_otp"
    gem "bcrypt", "~> 3.1.2"

    gem "omniauth"
    gem "omniauth-facebook"
    gem "omniauth-twitter"

    # Be an oauth provider
    gem "doorkeeper", git: "https://github.com/doorkeeper-gem/doorkeeper"

    # These are a couple of my favs, if I forget them I regret it
    gem 'strapless'
    gem 'awesome_print'

    group :development do
      # We want to be able to debug in development
      gem 'pry'
      # Keeping the database fields in the models and specs makes things easier
      gem 'annotate'
    end

    group :test do
      # Make our specs watchable with beautiful progress bar
      gem 'guard-rspec'
      gem 'fuubar'
    end

    group :test, :development do
      # Get specs involved in this process
      gem 'rspec-rails', '~> 3.0.0'
      gem "shoulda-matchers"
      gem "factory_girl_rails"
    end

    # For notifykit
    gem "mustache"
    gem "github-markup", require: "github/markup"
    gem "github-markdown"
    gem "redcarpet"


Once you are done, you need to bundle the gems for your version of ruby. This will automatically figure out all of the dependencies:

    bundle
    
> What if you are offline, working on an airplane desperately fighting bad wifi? You can install your gems from local sources using `bundle install --local`.
    
## Documentation    

Kill the README.rdoc:

    rm README.rdoc
    
long live the README.md:

    # Sample
    

    # Getting the code

    First things first, you need the repository:

        git clone git@github.com:YOURUSERNAME/sample.git

    Once you have that you'll want to switch to that folder. This project was built using
    Rails 4.0.x and Ruby 2.0.0-p247 Ruby versions are managed with rbenv and when switching to
    the folder you should receive a notice that you need to install ruby if it is missing.

    Once you have the code you should be able to bundle.
    
    # Database
    
    The database requires you to use postgres in development. Assuming you have this you should be able to migrate
    
        bundle exec rake db:create db:migrate db:seed
        
    For tests:
    
        bundle exec rake RAILS_ENV=test db:create db:migrate db:seed
    
    # Running locally
    
    To start the app locally you'll want to use foreman:
    
        foreman start
        
    # Specs
    
    Testing is done using RSpec. You can run this continuously using Guard:
    
        bundle exec guard

## Git and Source Control

You want to keep track of your files so you'll want to setup the repository.

First you need to create a new repository on Github (if you are using Github). You might want this to be a private repository if you have a paid account. You might want it to be part of an organization. I am skipping all of that. 

Once complete, you'll need to initialize the repo:

    git init
    git remote add origin git@github.com:YOURUSERNAME/sample.git

Setup the `.gitignore`. This will keep all of the user specific files (and sensitive files) out of the repo:

    *.rbc
    *.sassc
    .sass-cache
    capybara-*.html
    .rspec
    /.bundle
    /vendor/bundle
    /coverage/
    /spec/tmp/*
    /public/assets

    # Ignore logs and temp files
    /log/*
    /tmp/*

    # Ignore databases
    /db/*.sqlite3
    /db/*.sqlite3-journal

    # Ignore uploads
    /public/system/*

    # Ignore other junk that some crops up
    **.orig
    rerun.txt
    pickle-email-*.html

    # Ignore temp stuff
    designs
    samples
    TODO.md

    # Vagrant
    .vagrant

    # Chef
    encrypted_data_bag_secret
    chef/cookbooks
    
    # Ignore config (optional)
    config/application.yml
    
    # You may want to ignore .env if you are using DotEnv
    # If you do ignore, you should have a .env.example
    .env

    # In modern Rails you do not ignore database.yml 
    # or secrets.yml by default. But if you store secrets in them
    # you must not check them in to the repository
    # config/database.yml
    # config/secrets.yml


Make an initial commit and push it to the server:

    git add .
    git commit -a -m "Initial commit"
    git push -u origin master


## Setting up specs

You want to test things so you'll need to _install rspec_. Now really, you don't need to test anything, but if you show other Rubyists your code they will scoff. Apart from that, testing things (or speccing them) tends to help you think through your code, find bugs, and prevent regressions.

    rails g rspec:install

Modify the spec helper in spec/spec_helper.rb. By default the configuration is commented out using `=begin` to `=end`. The default configuration is good and you should remove the comment markers and use the settings provided. 

The default settings allow you to focus specs using `focus: true`, but it is helpful to add aliases to the `it` method:

    config.alias_example_to :fit, focus: true
    config.alias_example_to :pit, pending: true

In Rspec 3 there are Rails specific settings in `spec/rails_helper.rb`. To use Factory girl shortcuts you need to add the config option in the config block in the Rails spec helper:

    # Add factory girl
    config.include FactoryGirl::Syntax::Methods

### Options

When Rspec installs it adds a `.rspec` file that has the default options. By default warnings are on and this tends to create a lot of noise.

    --format Fuubar
    --color
    --require spec_helper
    --profile

If you want to see the warnings (you don't) you can add:

    --warnings
    
### Guard    
    
_Get a Guardfile_. Gaurd can watch your rails folder for changes and re-run the related tests:

    bundle exec guard init

Now, I don't know about you but I like to change my guard command like so:

    guard :rspec, cmd: 'bundle exec rspec' do
    
Now run guard:

    bundle exec guard
    
You might have a ton of warnings. When you run the RSpec 3 installer it adds a `.rspec` file to your Rails root that has `--warnings` turned on. You can turn these warnings off by removing that line.

## Foreman and running development

Foreman helps you start up all of your processes in a predefined way. In order to use Foreman you'll need a `.env` file:

    RACK_ENV=development
    PORT=3000

You can use this to easily change the default port for your local Rails server. You'll also want a `Procfile`:

    web:	bundle exec thin start -p $PORT -e $RACK_ENV

Now, instead of running `rails s` you can run:

    foreman start
    
You may need to install the `foreman` gem (or you can run it with `bundle exec foreman start`). Instead of running `rails console` directly, you should also use foreman:

    foreman run rails console
    
This will load your `.env` when setting up the console.

## Setting up settings

You will likely have some settings that you don't want stored in your repository and generally, moving secret keys around is a bad idea. On Heroku you can use ENV vars instead, and in fact on your own deployment you can do the same. Make sure that you have `.env` in your `.gitignore`. Then add the following to your `.env`:


    RACK_ENV=development
    PORT=3000
    SOME_API_SECRET=Your acccss key

Putting your secret keys and config keys into the `.env` will insure that they are not added to your repository and stored remotely. 
    
Because these environment variables are loaded you can use them throughout your program. However, you may want to use them directly in your `config/secrets.yml`.

    defaults: &defaults
      api_secret: <%= ENV["SOME_API_SECRET"] %>

    development:
      <<: *defaults
      secret_key_base: cbed3176b89ca67d77601a133480b6e2770e66c76e04c0b5b71fd7c4a84635485c314421034b134602e064496da738c9cbb56d64926ebadd6a883047da65a3d5

    test:
      <<: *defaults
      secret_key_base: 5c0a1d6fa5a9d104bc712d580f4f2ee26b46f1084ed77e2e055d85ba0e4021b6baff5087b67ee423ec46fbc789a83f266459321dd440da2e6836032335bb2c07

    production:
      <<: *defaults
      secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>

With your secrets created you can use these anywhere in your Rails application:

    Rails.application.secrets.api_secret

You can use the environment variable as a default value or embed it in each environment directly. It is also possible to have multiple `.env` files per environment. For example you could keep your development secrets in a `.env.development` file (remember to add this to your `.gitignore` if necessary). If you wanted to load this when running foreman you could do the following:

    foreman start -e .env,.env.development    
    

If you are using Heroku or a similar platform you'll want to push those keys. Here is a crazy one-liner:

    heroku config:set `cat .env`      
    
Overall this is not a very secure way to deal with your settings. If they live on your local machine unencrypted they are not safe and can be stolen. Also, this encourages passing the `.env` file around (never mail or skype this to another developer on your team). 

If you have a lot of setup in your `.env` that is required, it is helpful to make a `.env.example` that does not contain secrets so that other developers have a place to start. If the example file doesn't have secrets, it can be safely checked into your repository.

### Secret token      

Storing secrets in `config/secrets.yml` works really well and gives you a nice API for accessing these items. Typically this is where the `secret_key_base` is stored and in the development and test environments you can store these directly in the file. 

Your production `secret_key_base` should be kept very secret. With that secret, any attacked could decrypt remote session cookies and impersonate other users easily. 

> In fact, you should absolutely add a new key if you have already committed one (though this could impact currently logged in sessions).    

On Heroku, the `SECRET_KEY_BASE` environment variable is defined for you so that you don't need to manage the production secret locally.

If you want to manage this yourself (you should not), you can generate secret keys by using `rake secret`. 

Never store the secret key directly in `config/initializers/secret_token.rb` 

## Oh that database?

If you are using postgresql, make sure you have the gem `pg` in your Gemfile and bundle. A default `config/database.yml` should work fine. You may be ignoring the `database.yml` in your `.gitignore`. If so, you could create an example file but the modern approach is to check this file into your repository and rely on the environment to supply secrets (just like `config/secrets.yml`). To create an example:

    cp config/database.yml config/database.yml.example

Once you have the `database.yml` you can create the database

    bundle exec rake db:create db:migrate db:seed

If you need to clear your database, stop rails (and any testing) to disconnect and run:

    bundle exec rake db:reset
   
This clears all of the records and re-seeds. If you want to re-run migrations from the beginning:   

    bundle exec rake db:drop db:create db:migrate db:seed

You may need to update your test database:

    bundle exec rake RAILS_ENV=test rake db:migrate


