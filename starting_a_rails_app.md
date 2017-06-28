Starting a Rails app (v5.1)
===========================

This is only the basics. Getting your repository, your organization and getting some of the basic spec infrastructure setup.

## Ruby and Rails

_Get your ruby ready_. I use rbenv and ruby-build. There are a few other options available (rvm, etc.) but this is the one that has worked for me:

    brew upgrade ruby-build
    rbenv install -l

    # Install the newest
    rbenv install 2.5.1
    rbenv global 2.5.1
    gem install rake rails bundler foreman mailcatcher   
    
_Install your basic gems_. If you have just installed a new version of ruby you might need to `gem install bundler rake rails` 

_Initialize the project_. I have a folder called projects which I can access via /projects. This is just a symlink to ~/Projects:

    cd /projects

Within my projects folder I will _start the new Rails app_ and this will create an application folder and generate all of the initial files:

    rails new sample -d postgresql --skip-test-unit --skip-turbolinks
    cd sample

> Note: if you are creating an app and plan to deploy to heroku you probably want to use postgres as your database. If you are using another platform you may not want postgres. In that case you should just create the app without that set: `rails new sample --skip-test-unit`

In this case my application folder is /projects/sample. This is my "Rails Root" path for my application (on my development machine).

You want to use the latest ruby there (this creates a .ruby-version file):

    rbenv local 2.5.1

_Remove the test folder_. you want to use specs instead (at least I do, you might like tests).

    rm -rf test
    
## Gems

Setup the `Gemfile`. This sample Gemfile has quite a few helpful defaults, but you should compare it with the generated Gemfile and pick and choose:

```ruby
# You should specify the ruby version
ruby "2.5.1"

source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end


# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 5.1.0'
# Use postgresql as the database for Active Record
gem 'pg', '~> 0.18'
# Use Puma as the app server
gem 'puma', '~> 3.7'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 5.0'
# Use Uglifier as compressor for JavaScript assets
gem 'uglifier', '>= 1.3.0'
# See https://github.com/rails/execjs#readme for more supported runtimes
# gem 'therubyracer', platforms: :ruby

# JSON
gem 'active_model_serializers'

# Processes
gem 'foreman'
gem 'sidekiq'

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

# These are a couple of my favs, if I forget them I regret it
gem 'strapless'
gem 'awesome_print'

# For tokens
gem 'hashids'

group :development, :test do
  # Keeping the database fields in the models and specs makes things easier
  gem 'annotate'
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  # Get specs involved in this process
  gem 'rspec-rails', '>= 3.5.0'
  gem 'shoulda-matchers', require: false
  gem 'factory_girl_rails'
end

group :development do
  # Access an IRB console on exception pages or by using <%= console %> anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '>= 3.0.5', '< 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

group :test do
  # Make our specs watchable with beautiful progress bar
  gem 'guard-rspec', require: false
  # Better formatting of rspec
  gem 'fuubar'
end
```

Once you are done, you need to bundle the gems for your version of ruby. This will automatically figure out all of the dependencies:

    bundle
    
> What if you are offline, working on an airplane desperately fighting bad wifi? You can install your gems from local sources using `bundle install --local`.
    
## Documentation    

Long live the README.md:

    # Sample
    
    A description of your application.

    ## Getting the code

    First things first, you need the repository:

        git clone git@github.com:YOURUSERNAME/sample.git

    Once you have that you'll want to switch to that folder. This project was built using
    Rails 5.1.x and Ruby 2.4.x Ruby versions are managed with rbenv and when switching to
    the folder you should receive a notice that you need to install ruby if it is missing.

    Once you have the code you should be able to bundle.
    
    ## Environment

    Make a `.env` file:

        RACK_ENV=development
        PORT=3000
        DOMAIN=sample.dev

    ## Mail

    You need to send mails locally, to do this use `mailcatcher`:

        gem install mailcatcher

    Then run it:

       mailcatcher

    Once you have sent mails you can view them at [http://localhost:1080](http://localhost:1080)

    ## Database
    
    The database requires you to use postgres in development. Assuming you have this you should be able to migrate
    
        bundle exec rake db:create db:migrate
        
    For tests:
    
        bundle exec rake RAILS_ENV=test db:create db:migrate
        
    For seeding:
    
        foreman run bundle exec rake db:seed
    
      
    ## Running locally
    
    To start the app locally you'll want to use foreman:
    
        foreman start
        
    ## Specs
    
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
    !/log/.keep
    !/tmp/.keep

    # Ignore Byebug command history file.
    .byebug_history

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
    spec/examples.txt

    # Vagrant
    .vagrant

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

## Config

If you are sending email then you can use [Mailcatcher](http://mailcatcher.me) locally. This allows you to catch all email being sent from Rails and view it.

> To install Mailcatcher just `gem install mailcatcher`. Then run `mailcatcher`. When running you can open a browser to [see the mails](http://localhost:1080).
    
You'll need to configure mailcatcher in `config/environments/development.rb`:
      
    # Mailcatcher
    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025, :domain => Rails.application.secrets.domain }
    config.action_mailer.raise_delivery_errors = true
    config.action_mailer.perform_deliveries = true

This might be outdated:

    # Fix the Safari 304 not modified bug (https://bugs.webkit.org/show_bug.cgi?id=32829)
    config.middleware.delete Rack::ETag

And in `config/application.rb`:

    config.action_mailer.default_url_options = { host: Rails.application.secrets.domain }
    config.action_mailer.asset_host = Rails.application.secrets.domain


## Setting up specs

You want to test things so you'll need to _install rspec_. Now really, you don't need to test anything, but if you show other Rubyists your code they will scoff. Apart from that, testing things (or speccing them) tends to help you think through your code, find bugs, and prevent regressions.

    rails g rspec:install

Modify the spec helper in `spec/spec_helper.rb`. By default the configuration is commented out using `=begin` to `=end`. The default configuration is good and you should remove the comment markers and use the settings provided. 

The default settings allow you to focus specs using `focus: true`, but it is helpful to add aliases to the `it` method. You can add these into a helper like `spec/support/aliases.rb`:

    RSpec.configure do |config|
      config.alias_example_to :fit, focus: true
      config.alias_example_to :pit, pending: true
    end

In Rspec 3 there are Rails specific settings in `spec/rails_helper.rb`. To use Factory girl shortcuts you need to add the config option in the config block in the Rails spec helper:

    # Add factory girl
    RSpec.configure do |config|
      config.include FactoryGirl::Syntax::Methods
    end

Again, you can add this into a helper called `spec/support/factory_girl.rb`. 

In order to load all of the files in `support` you'll need to require them from the `spec/rails_helper.rb`. Make sure the following line is not commented out:

     Dir[Rails.root.join("spec/support/**/*.rb")].each { |f| require f }
     
If you are using `Shoulda` then in `rails_helper.rb` add the following: 

    # Add additional requires below this line. Rails is not loaded until this point!
    require 'shoulda/matchers'

    Shoulda::Matchers.configure do |config|
      config.integrate do |with|
        with.test_framework :rspec
        with.library :rails
      end
    end
  

### Globally calling `describe`

You may also want to add the ability to use the `describe` DSL globally (no longer the default). You can add the following to the end of the config block in the `spec_helper.rb`:

    # Allow specs to begin with `describe` instead of `RSpec.describe`
    config.expose_dsl_globally = true


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

    web:    bundle exec puma -p $PORT -e $RACK_ENV -C config/puma.rb
    worker: bundle exec sidekiq

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
      domain: 'example.com'
      domain_friendly: 'example.com'
      company_name: 'Example, Inc.'
      company_email_legal: 'legal@example.com'
      company_address: '231 S Buena Vista, Redlands, CA 92373'
      from_email: <%= ENV["EMAIL_FROM_ADDRESS"] || "Example.com <support@example.com>" %>
      api_secret: <%= ENV["SOME_API_SECRET"] %>

    development:
      <<: *defaults
      domain: 'localhost:3000'
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

    foreman start -e .env,.    env.development    
    

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


## Money in Rails

Don't represent money using floating point numbers. Use the [monetize](https://github.com/RubyMoney/monetize) gem and the [money-rails](https://github.com/RubyMoney/money-rails) gem:

    gem 'monetize'
    gem 'money-rails'

And setup the initializer:

    rails g money_rails:initializer 
    
Store your fields as integers with cents (you can customize this with currencies using the `t.money` method):

    t.integer :price_cents, default: 0    
    
Then in your models:

    class Product < ActiveRecord::Base
      monetize :price_cents # allow_nil: true
    end
    
## Bootstrap, fonts, views

#### `app/views/application.html.erb`

```
<!DOCTYPE html>
<!--[if IE 8]>         <html lang="en" class="ie ie8 lt-ie9"> <![endif]-->
<!--[if IE 9]>         <html lang="en" class="ie ie9">        <![endif]-->
<!--[if gt IE 9]><!--> <html lang="en">                       <!--<![endif]-->
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">
    <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
    <link rel="icon" type="image/png" href="/favicon-32x32.png" sizes="32x32">
    <link rel="icon" type="image/png" href="/favicon-16x16.png" sizes="16x16">
    <link rel="manifest" href="/manifest.json">
    <link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
    <meta name="theme-color" content="#ffffff">
    <%= csrf_meta_tags %>
    <title><%= content_for(:title) || "SAMPLE TITLE" %></title>
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
    <%= stylesheet_link_tag "application", :media => "all" %>
    <%= yield :stylesheets %>
    <link href="https://fonts.googleapis.com/css?family=Abril+Fatface|Nunito+Sans" rel="stylesheet">
    <script src="https://use.fontawesome.com/c34f8e1c39.js"></script>
    <% if Rails.env.production? %>
    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
      ga('create', 'SAMPLE GOOGLE ANALYTICS', 'https://sellers.community');
      ga('send', 'pageview');
    </script>
    <% end %>
  </head>
  <body class="<%= "#{current_controller}-#{current_action}" %> <%= content_for :classes %> <%= 'flash' if flash.present? %>">
    <%= render 'layouts/nav' %>

    <% if flash.present? %>
    <div class="container-fluid bg-success">
      <div class="row">
        <div class="container">
          <div class="row">
            <div class="col-lg-12">
              <div id="flash">
                <% flash.each do |type, msg| %>
                <p class="flash <%= type %>"><%= msg %></p>
                <% end %>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <% end %>

    <%= yield %>
    <%= render 'layouts/footer' %>
    <div id="lightbox-background"></div>
  </body>

  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
  <script src="//ajax.googleapis.com/ajax/libs/jqueryui/1.11.2/jquery-ui.min.js"></script>
  <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
  <%= javascript_include_tag "application" %>
  <%= yield :javascripts %>
</html>
```    

#### `app/views/layouts/_nav.html.erb`

```
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="navbar-header">
        <a class="navbar-brand" href="/">
          <%= image_tag "sellers.png", style: "max-width:18px;display:inline-block" %><%= image_tag "sellers-community.png", style: "margin-left:16px;max-height:12px;display:inline-block;" %>
        </a>
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
      </div>
      <div id="navbar" class="collapse navbar-collapse">
        <ul class="nav navbar-nav">
          <li class="<%= 'active' if request.path == '/faq' %>"><a href="/faq">Questions &amp; Answers</a></li>
          <% if logged_in? %>
            <li class="<%= 'active' if request.path == '/logout' %>"><a href="/logout">Logout</a></li>
            <li class="<%= 'active' if request.path == '/account' %>"><a href="/account"><span style="font-weight:700"><%= current_user.email %></span></a></li>
          <% else %>
            <li class="<%= 'active' if request.path == '/login' %>"><a href="/login">Login</a></li>
            <li class="<%= 'active' if request.path == '/' %>"><a href="/signup"><button type="button" class="btn btn-primary"><span>Register now</span></button></a></li>
          <% end %>
        </ul>
      </div>
      <div id="navsep"><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span></div>
    </nav>
```

#### `app/views/layouts/_footer.html.erb`

```
<footer class="footer">
  <div class="row">
    <div class="col-lg-12">
      <div class="copyright text-muted pull-right">
        <span>Copyright &#169; <%= Time.now.year %>, SAMPLE. All rights reserved.</span>
        <span class="policies">
          <a href="/privacy">Privacy Policy</a> |
          <a href="/terms">Terms of Service</a>
        </span>
      </div>
    </div>
  </div>
</footer>
```

> Something about css... rename application.css to application.scss (if you want to have the @import work


You'll need some helpers in `/app/helpers/application_helper.rb`

```
module ApplicationHelper
  def current_controller
    params[:controller]
  end

  def current_action
    params[:action]
  end

  def logged_in?
    false
  end
end
```

## Welcome controller

```
rails g controller pages
```

```
Running via Spring preloader in process 25291
      create  app/controllers/pages_controller.rb
      invoke  erb
      create    app/views/pages
      invoke  rspec
      create    spec/controllers/pages_controller_spec.rb
      invoke  helper
      create    app/helpers/pages_helper.rb
      invoke    rspec
      create      spec/helpers/pages_helper_spec.rb
      invoke  assets
      invoke    js
      create      app/assets/javascripts/pages.js
      invoke    scss
      create      app/assets/stylesheets/pages.scss
```

Now add a route (in `config/routes.rb`):

```
  root 'pages#welcome'
```

Now add the corresponding view:

`/app/views/pages/welcome.html.erb`

```
<div id="welcome">
  <div class="container">
    <div class="row">
      <div class="col-md-6 page-header">
        <h1>Welcome</h1>
        <p class="lead">
          You might see something here you love.
        </p>
        <p>
          No, seriously.
        </p>
      </div>
      <div class="col-md-6 page-header">
      </div>
      <div class="col-md-12">
      </div>
    </div>
  </div>
</div>
```

#### Forms

Add the following to your `Gemfile`:

```
# Forms
gem 'bootstrap_form'
```

and then `bundle`.

For your signup form you can use this:

```
<div id="contact">
  <div class="container">
    <div class="inner">
      <div class="row">
        <div class="col-lg-8 col-md-8 col-sm-8">
          <div class="page-header">
            <h1>Create an account</h1>
            <p class="lead">Make an account.</p>
            <p>Already have an account? <a href="/login">Log in</a>.</p>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-lg-6 col-md-6 col-sm-6">
          <% if @signup.errors.any? %>
            <div id="error_explanation">
              <div class="alert alert-error">
                The form contains <%= pluralize(@signup.errors.count, "error") %>.
              </div>
              <ul>
              <% @signup.errors.full_messages.each do |msg| %>
                <li>* <%= msg %></li>
              <% end %>
              </ul>
            </div>
          <% end %>
        </div>
      </div>
      <div class="row">
        <div class="col-lg-5 col-md-5 col-sm-5 required">
          <%= bootstrap_form_for @signup, as: :signup, url: signup_path do |f| %>
            <%= f.text_field "first_name", label: "First name", placeholder: "First name"  %>
            <%= f.text_field "last_name", label: "Last name", placeholder: "Last name"  %>
            <%= f.telephone_field "phone_number", placeholder: "555-555-5555" %>
            <%= f.email_field "email", placeholder: "Email address" %>
            <%= f.time_zone_select 'time_zone', ActiveSupport::TimeZone.us_zones, :default => "Pacific Time (US & Canada)" %>
            <br>
            <%= f.password_field "password", placeholder: "Password" %>
            <%= f.password_field "password_confirmation", placeholder: "Re-type password" %>
            <div class="form-group">
              <label for="remember_me" class="control-label"><%= check_box_tag :remember_me, "1", true %>
              Keep me signed in on this computer</label>
            </div>
            <br>
            <%= f.primary "Sign up" %>
          <% end %>
        </div>
      </div>
    </div>
  </div>
</div>
```


## Favicon

http://realfavicongenerator.net/

## Cloudflare

## Auditing and Papertrail

Sometimes when you make changes you want a history of what changed. PaperTrail offers you an audit log:

https://github.com/airblade/paper_trail

Add PaperTrail to your Gemfile.

```ruby
gem 'paper_trail'
```

Add a `versions` table to your database and an initializer file for configuration:

```bash
bundle exec rails generate paper_trail:install --with-associations
bundle exec rake db:migrate
```

If using `rails_admin`, you must enable the experimental Associations feature. For more information on this generator, see section 5.c. Generators.

Add `has_paper_trail` to the models you want to track.

```ruby
class Widget < ActiveRecord::Base
  has_paper_trail
end
```

If your controllers have a `current_user` method, you can easily track who is responsible for changes by adding a controller callback.

```ruby
class ApplicationController
  before_action :set_paper_trail_whodunnit
end
```

## Admin

https://github.com/sferik/rails_admin

1. On your gemfile: gem 'rails_admin', '~> 1.2'
2. Run bundle install
3. Run rails g rails_admin:install
4. Provide a namespace for the routes when asked

Start a server rails s and administer your data at /admin. (if you chose default namespace: /admin)

You'll need to edit the config at `config/initializers/rails_admin.rb`. First, add in a controller reference:

```ruby
## Controller
config.parent_controller = "ApplicationController"
```

To authenticate:

```ruby
## Auth
config.current_user_method(&:current_user)
```
  
To authorize, add a new field to your User model called `admin` with a default `false`.

```ruby
config.authorize_with do
  redirect_to main_app.root_path unless current_user.admin?
end
```

Make sure you active the `PaperTrail` support:

```ruby
config.audit_with :paper_trail, 'User', 'PaperTrail::Version' # PaperTrail >= 3.0.0
```
