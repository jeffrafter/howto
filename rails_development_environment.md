# Rails development environment


-- Lots to do here

# Update rbenv to get the latest ruby

You might have installed rbenv/ruby-build in one of three ways (on OSX):

As a git repo (old method):

```
cd ~/.rbenv
git pull
rbenv install -l
rbenv install 2.1.1
```
    
Installed via Homebrew:

```
brew update
brew upgrade rbenv ruby-build
rbenv install -l
rbenv install 2.1.1
```

Installed with ruby-build as a plugin (currently the recommended way):

```
cd ~/.rbenv/plugins/ruby-build
git pull
rbenv install -l
rbenv install 2.1.1
```
