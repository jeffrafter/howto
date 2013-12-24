# Git

Lots of folks have opinions on how you should use Git. I have used a lot of the different methods and prefer the method that Scott Chacon writes about [Github Flow](http://scottchacon.com/2011/08/31/github-flow.html).

He outlines it as follows:

1. Anything in the master branch is deployable
2. To work on something new, create a descriptively named branch off of master (ie: new-oauth2-scopes).
3. Commit to that branch locally and regularly push your work to the same named branch on the server.
4. When you need feedback or help, or you think the branch is ready for merging, open a pull request.
5. After someone else has reviewed and signed off on the feature, you can merge it into master.
6. Once it is merged and pushed to ‘master’, you can and should deploy immediately.

## Usage

Go to your Rails root folder:

    git status
   
You should see the status of your repository which might look something like this:

    # On branch master
    nothing to commit, working directory clean

At this point your master branch is up to date and you are ready to start a new feature:

    git checkout -b my-new-feature

This will make a copy of master and checkout a new branch that you can modify without hurting the other branches (even though you haven't changed folders). Notice the `-b` option here tells git to **create** the branch. 

If your prompt shows the current branch (it should!) then you'll see something like:

    /projects/sample [my-new-feature] $
    
Either way, checking the status again shows you where you are:

    git status
    
Where you'll see:

    # On branch my-new-feature
    nothing to commit, working directory clean        
    
Now you can start working away, making changes and committing them. If you want to push them up to the remote server (for instance, Github) just:

    git push origin my-new-feature
    
At this point others can `git fetch` to get the branches and `git checkout my-new-feature` to see your changes.

### What if `master` changes before I push?

If master is updated (it probably will be) there are a couple of options. You can merge in the changes or rebase. I prefer to **rebase**. Commit any changes on your current branch. Then go back to the `master` branch

    git checkout master
    
Get the latest (also using a rebase):

    git pull --rebase origin master
    
Now, this rebase is probably unnecessary. You shouldn't be making any changes on the master branch anyway. So this should pull down the latest code without issues. Now jump back to your branch:

    git checkout my-new-feature
    
Now lets rebase on top of master:

    git rebase master
    
At this point git will take all of the commits from your current branch and re-play them on top of the latest master. So if you added a file or deleted a line or replaced an image, git will do that again but will pretend that you just started doing it with the latest code. 

This doesn't always go perfect. What if you made a change to a file that has now been deleted? When git tries to re-play that change it will see that the file is missing and give you a **conflict**.

You need to fix the conflict by deciding what the current state should be. You can keep the file deleted, add the file back in or fix the conflict lines (which will show in the files with the HEAD and version specific changes). Once you make the change add the file (here I cheat, by adding everything):

    git add .
    
Then continue the rebase

    git rebase --i continue
    
Now, technically, when you do this you are rewriting history. You didn't _actually_ make these commits after everything in master, you are just rewriting history to make it cleaner. This is a good thing, but it is also a bad thing. If you try to push up the changes from your freshly rebased branch you will get a conflict (if you have already pushed before). You can get around this by forcing the push:

    git push -f origin my-new-branch
    
But by doing so you rewrite the shared history, which is even worse! Anyone who has fetched your branch will probably get a conflict on their end. If they are simultaneously working on the same branch it can become very confusing. If they have pushed changes that you don't have you can overwrite their changes.

**You should never-ever force push to master**. 

## Pull Requests

Once you have finished all of the changes you should have them reviewed. Generally this is done with a Pull Request. On Github, switch to your branch and hit the pull request button. You should be able to create the pull request and someone on the team can review it. If you make a change and re-push the branch, Github will automatically update the pull request.

Once everything is reviewed and looks great (see also, continuous integration), the branch can be merged to master and the remote branch can be deleted. 

When the branch is merged it is actually a merge: not a rebase. This is generally okay as the history would otherwise get very confusing as multiple teams members create code simultaneously.

Once you have merged you should delete the local copy of the branch as well:

    git checkout master
    git branch -D my-new-feature

And get the latest master:

    git pull --rebase origin master
    
Now you are ready to begin working on a new feature.

It should be noted that at this point the branch should be deployed (see also, deployment) and any feature tracking (pivotal, etc). Should be updated. There are ways to integrate these directly into the github flow. 