Starting a Rails app
====================

This is only the basics. Getting your repository, your organization and getting some of the basic spec infrastructure setup.

## Ruby and Rails

_Get your ruby ready_. I use rbenv and ruby-build. There are a few other options available (rvm, etc.) but this is the one that has worked for me:

    brew upgrade ruby-build
    rbenv install 2.0.0-p353
    rbenv global 2.0.0-p353
    gem install rails

_Initialize the project_. I have a folder called projects which I can access via /projects. This is just
a symlink to ~/Projects:

    cd /projects

Within my projects folder I will _start the new Rails app_ and this will create an application folder and generate all of the initial files:

    rails new sample
    cd sample

In this case my application folder is /projects/sample. This is my "Rails Root" path for my application (on my development machine).

You want to use the latest ruby there (this creates a .ruby-version file):

    rbenv local 2.0.0-p353

_Remove the test folder_. you want to use specs instead.

    rm -rf test
    
## Config

Setup the most basic config/application.rb. By default you only need the
following two options:

    # Set the test framework to rspec
    config.test_framework = :rspec

    # Set the default encoding
    config.encoding = "utf-8"

Setup the `Gemfile`. Grab the sample Gemfile gist as it has quite a few helpful defaults:

    curl https://raw.github.com/jeffrafter/sample/941e97b8f6a4e86c119de3fd6745ae452c9efe1b/Gemfile -o Gemfile

Once you are done, you need to bundle the gems for your version of ruby. This will automatically figure out all of the dependencies:

    bundle


Kill the README.rdoc, long live the README.md

    rm README.rdoc
    curl https://raw.github.com/jeffrafter/sample/941e97b8f6a4e86c119de3fd6745ae452c9efe1b/README.md -o README.md

## Git and Source Control

You want to keep track of your files so you'll want to setup the repository.

First you need to create a new repository on Github (if you are using Github). You might want this to be a private repository if you have a paid account. You might want it to be part of an organization. I am skipping all of that. 

Once complete, you'll need to initialize the repo:

    git init
    git remote add origin git@github.com:YOURUSERNAME/sample.git

Setup the `.gitignore`. This will keep all of the user specific files (and sensitive files) out of the repo:

    curl https://raw.github.com/jeffrafter/sample/941e97b8f6a4e86c119de3fd6745ae452c9efe1b/.gitignore -o .gitignore


Make an initial commit and push it to the server:

    git add .
    git commit -a -m "Initial commit"
    git push -u origin master


## Setting up specs

You want to test things so you'll need to _install rspec_. Now really, you don't need to test anything, but if you show other Rubyists your code they will scoff. Apart from that, testing things (or speccing them) tends to help you think through your code, find bugs, and prevent regressions.

    rails g rspec:install

Modify the spec helper in spec/spec_helper.rb. You need to add the config option in the config block:

    # Add factory girl
    config.include FactoryGirl::Syntax::Methods

_Get a Guardfile_. Gaurd can watch your rails folder for changes and re-run the related tests:

    bundle exec guard init

Now, I don't know about you but I like to change my guard command like so:

    guard :rspec, :cli => '--color --format Fuubar --profile', :all_on_start => false, :all_after_pass => false do

Now run guard:

    bundle exec guard

