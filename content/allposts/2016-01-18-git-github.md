---
title: Github and Version Control
subtitle: do you git it?
date: '2016-01-18'
slug: github-git
---

# Git and Github usage

One of the most useful skills to have is the ability to work and collaborate
with other people on a project. Git, and Github is usually the tool of choice
for most programmers these days, although in the past I have actually used
other versioning and control options, like SVN. What Git allows you to do is
be able to work independently on a project, say for example, creating a
feature for the application, and then when you are done, you can submit that
feature to be added to the master source code. Also, many people use Github to
show repositories of their projects, and code, so that for anybody looking,
they can see how a person's coding style looks like.

# Useful commands

I've been using Github as a place to put my code for some time now, but it
wasn't until recently that I used it as a place to collaborate with somebody
on a project. The usage and commands of Git become quite different once you're
actually using it as a tool to manage different snapshots of code. For
example, my entire website is actually handled by Github, and the process of
writing this blog and then putting it onto Github is:

```
    middleman article "Github collaboration"
    middleman deploy
```

What has happened is that locally I have created a new blog file, and then
when I am done writing `middleman deploy` _pushes_ that local code onto the
Github servers and into my repo, and tada, the website gets updated. And for
code, my process is similar: 1\. make write code 2\. `add` to Git repo 3\.
`commit` code 4\. `push` code to Github servers

## Branching and Pull requests

But for collaboration, the true strength of Git shines through. The general
idea is that you have a master branch of code, where everything is perfect and
runs properly, and then everybody else can create various other branches. The
idea is to keep changes to the master branch small and manageable, so that if
things do break, you can revert.

To start off from the master branch, you run

```
    git checkout -b "adding-maps-feature"
```

which will make a new branch called `adding-maps-feature` and then place you
within that branch. Then here you can add whatever code you want to implement
that new feature. Once you are done, you run

```
    git add <whatever files you changed/added for the feature>
    git commit -m 'adding maps'
    git push --set-upstream origin adding-maps-feature
```

Those commands will add the changes to the branch with a commit message, and
then git push will push the changes to Github. Once you are all done this, you
can open a `pull request` with that branch on Github. A pull request will
allow your collaborators to review the changes in the code you submitted, and
then if they deem that everything is ok, that branch can be merged into the
master branch. The pull request also allows for collaborators to make comments
and suggestions on the changes.

Once the merge is complete, you are free to delete that branch and then work
on a different one. But one thing to remember is to change back to the master
branch within your local system with:

```
    git checkout master
    git pull
```

This will pull all the changes from the master branch on Github back to your
local system so you can now work on a new branch/feature.

## Multiple branches

It is very common for multiple branches, and multiple pull requests to exist
at one time in a project. So to manage a constantly changing codebase, it is
good to pull and merge local branches before pushing. To do this within the
current branch, run:

```
    git checkout master
    git pull
    git checkout - 
    git merge master
```

The `git checkout -` command returns you to your previous branch, and now it
will contain the most recent master code. Then when your branch is merged into
master, there won't be any conflicts.

Those would be the most used commands in my project, although there are other
commands that are useful for when conflicts exist, but that is a bit more
advanced.
