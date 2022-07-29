---
layout: post
title: Git… My new favourite SVN client
date: '2011-02-08T13:03:00+00:00'
tags:
- subversion
- svn
- git
- github
- version control
- programming
tumblr_url: https://richbs.org/post/3180536011/git-my-new-favourite-svn-client
---
My friend and former manager [Simon Jones](http://simonrjones.net/), Technical Director of the talents at [Studio 24](http://studio24.net/) challenged me to post my experiences with Git. So, I have cobbled together the commands and concepts that I’ve found most useful in transitioning from Subversion. Here goes, Si!

## Intro

[Git](http://git-scm.com/) is fast, unfussy and seems beautfully adapted to my workflow. Subversion is centralised, slow and usually makes me feel a bit clumsy. However, SVN remains a sensible choice for smaller dev teams within a hierarchical organisation due to its ubiquity and bureaucracy. I am delighted to discover that I can use Git as an interface to my SVN repositories. Git is now my favourite SVN client, especially when it comes to branching and merging.

If you’re prepared to discard your existing Subversion client and use Git exclusively (on a project-by-project basis), your SVN-using colleagues won’t see any difference in your commits to the Subversion repository and you too can be smug when enjoying the benefits of Git locally.

## Mental Notes

A couple of conceptual differences I needed to overcome.

1. Your local Git repository has the complete history of your project, meaning you have access to every changeset and version even when offline.
2. I usually maintain one Subversion repository with several directories; one directory per project. With Git, it’s one repository per project. You will likely map your local Git repo to a directory on your SVN server.

## A cheat sheet of sorts

Sensible prerequisites for these commands are a local installation of Subversion and Git, probably in that order.

### “Checking out”

With `git svn clone`, you tell Git to suck down every changeset from Subversion and create a local repo. Note this is not an svn checkout; regular svn won’t know what to do in this local directory.

The following command assumes that your svn project directories follow the usual trunk, branches, tags structure. Feel free to adjust the args to match your house style.

    # clone project from your Subversion repository
    git svn clone svn://server.name/svnrepo/project/directory -T trunk -b branches -t tags local_dir

This will take a little while and could copy quite a lot of data, so hop into the local directory that has been created and run Git’s excellent compression.

    # repack it
    git repack -a -d -f

You might have [created a blank repo on Github](http://help.github.com/creating-a-repo/) for your project. You can manage both SVN and Github from within your local repo.

    # link this repo to github
    git remote add origin git@github.com:username/project.git
    
    # push svn trunk to github master
    git push origin master

The Github stuff is optional but `master` refers to your local copy of SVN trunk. Now, back to more local Git/SVN integration.

### Branching

    # check which branches are available on svn
    git branch -a
    
    # create a new branch on the svn server (the -n switch does a dry run, remove it when ready)
    git svn branch -n git-test -m "branch for messing"
    
    # create a local git-test branch pointed at remote svn branch
    git checkout --track -b local/git-test remotes/git-test
    
    # check it's pointed at the right branch on svn server
    git svn info
    
    # Optional: push this branch to github (or have a dry run at it—remove --dry-run to do it for real)
    git push --dry-run origin git-test:svn/git-test
    
    # make lots of granular code changes and commit to your local Git repo freely and often
    git status
    git add changed/files.*
    git commit -m "committing small but perfectly formed changeset"

Once you have a sensible bunch of local commits, you can commit all those changesets (with messages included) to Subversion.

    # similar effect to "svn commit"
    git svn dcommit

### Merging

After a few commits to your branch you will want to merge into master locally and commit that to svn trunk.

    # fetch changes from SVN if other coders are active in your repo, e.g. other branches
    git svn fetch
    
    # switch to master
    git checkout master
    
    # make sure master is pegged to trunk
    git reset --hard remotes/trunk
    
    # rebase is the "svn update" equivalent. Run it on trunk.
    git svn rebase
    
    # merge in changes from the branch we created earlier (squashing all changesets into one fat one)
    git merge --squash local/git-test

^^^^_Look!_^^^^ no svn log, no specifying revision numbers. Git “just knows” what you want to merge.

Clean up after the merge, adding files to master and resolve conflicts if necessary (unlikely).

    git add new/file.py
    
    # commit merged result to local master
    git commit -m "Merging git-test branch."
    
    # commit merge to svn trunk
    git svn dcommit

So, now you can play with the cool kids on github whilst fulfilling your VC obligations at work.

And remember, your local Git repo has EVERYTHING, so get a graphical Git browser like [gitx](http://gitx.frim.nl/) on the Mac or use [gitk](http://www.kernel.org/pub/software/scm/git/docs/gitk.html) and enjoy surfing your favourite changesets from yesteryear.

