---
title: Git-SVN
date: 2013-01-16
tags: [git]
---

If you work on a project that's been around for a while, chances are it 
might still be using SVN for version control. Even if you can't get 
buy-in from Management or Ops to move to Git, you can still get most of 
the benefits by learning the ins and outs of the `git svn` sub command.

# Initial Clone

```
$ git svn clone --stdlayout svn://svn.example.com/project
```

Using `--stdlayout` tells Git that your project follows the common 
layout for trunk, branches, and tags. This allows you to leave off the 
`/trunk` when cloning and is important for the branching strategy I'll 
mention later.

Alternatively, you could only clone trunk (and have to separately clone 
every branch or tag you work with) or use separate options to specify 
the directory structure of your project's trunk, branches, and tags.

## Commits and Branching

Now you can work with the repo as if it were a normal Git repo: commit 
at will, create and merge local branches, etc. When you want to sync 
remote changes on trunk to your local master, run:

```
$ git svn rebase
```

If you're unfamiliar with rebasing, you should spend some time reading 
up on the Git concept itself. Conceptually, Git will remove any local 
changes you've made, sync master with trunk, then reapply your changes 
over top of the new master.

When you're satisfied with your local changeset, you can publish your 
work back to trunk with:

```
$ git svn dcommit
```

This will make your local commits one-for-one as SVN commits on trunk. 
It uses these commits and their messages as-is, so make sure `git log` 
shows exactly what you want to publish before you execute this.

{{< well >}}
Both of these commands require a clean working directory. Later, you'll 
see a shortcut for using `git stash` to get around this limitation.
{{< /well >}}

Branches which you create with a simple `git checkout -b foo` will be 
local-only branches. They have no relationship with any SVN branches and 
you must not try to `dcommit` or `rebase` when on these branches.

Master obviously has a relationship to trunk and you can checkout and 
work with other remote SVN branches similarly with:

```
$ git checkout -b foo_local foo
```

Where `foo` is an SVN branch. Adding the `_local` suffix is my 
convention, you can use anything that's unambiguous.

When on this branch, any `rebase` or `dcommit` you do will be 
interacting with that SVN remote branch.

If branches are added in SVN since your initial clone, you won't be able 
to check them out until you let your local repo know about them:

```
$ git svn fetch
```

{{< well >}}
This effectively does a `rebase` of all branches as part of the update.
{{< /well >}}

At any time, you can a list of all remote branches with:

```
$ git branch -r
```

## Example Workflow

For non-trivial project work, I'll typically make a local topic-branch. 
This allows me to jump back onto master and re-branch if I need to shift 
gears. It also gives me a chance to fiddle with the commits during the 
merge to master before I publish it out to SVN.

The simplest case of this would be using a "squash" merge:

```
$ git checkout -b fix-123
$ ...
$ git commit -am 'fix this thing'
$ ...
$ git commit -am 'fix that other thing'
$ ...
$ git commit -am 'change how I did this or that'
$ ...
$ git commit -am 'add test coverage ;)'
```

With a branch full of messy commits but a nice clean change-set, I can 
make all that as one SVN commit:

```
$ git checkout master
$ git merge --squash fix-123
$ git commit -m 'Fix Issue #123 ...'
$ git svn dcommit
```

There are more complicated rewrites and rebasings you could do, but this 
is the workflow I find myself using most often.

## Aliases

I'm still coming up with these as needed, but here are a few aliases 
that should make things easier. Just dump them into a `.gitconfig`.

```
[alias]
  sha = svn log --show-commit --oneline -r
  spull = !git stash && git svn rebase && git stash apply
  spush = !git stash && git svn dcommit && git stash apply
```

The latter two should be fairly obvious, but the first allows you to 
print the SHA for a given revision; very useful when talking to team 
members who use plain SVN:

```
$ git sha 34961
r34961 | b6a9f38 | Fix the thingy #123
```
