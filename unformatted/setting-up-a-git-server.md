
# Setting up a git server

See [http://gitolite.com/gitolite/gitolite.html#install](http://gitolite.com/gitolite/gitolite.html#install)

You need git:

    apt-get install git

Add a user (using `sudo` or as root):

    useradd -D -s /bin/bash
    useradd git
    usermod -a -G ssh-user git
    mkdir /home/git
    mkdir /home/git/.ssh
    chmod 700 /home/git/.ssh
    chown git:git /home/git -R

Switch to the git user and install:

    su - git
    mkdir -p /home/git/bin
    git clone git://github.com/sitaramc/gitolite
    gitolite/install -ln /home/git/bin        

You'll need your public key to setup the administrator account. From your local you can run `cat ~/.ssh/id_rsa.pub | pbcopy`. Then on the server:

    touch YOURNAME.pub
    nano YOURNAME.pub
    gitolite setup -pk YOURNAME.pub

You are setup. From your local:

   git clone git@YOURHOST:gitolite-admin

Now you can add new keys and you can configure the repos in the conf.
