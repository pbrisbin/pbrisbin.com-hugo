---
title: Vagrant
date: 2012-08-30
tags:
---

Despite the enormous increase in popularity lately, it seems some people 
still don't know about this project or how it can improve your 
development work-flow.

So, what is it and why should you care?

Vagrant is a convenience tool for doing Virtual Machine based 
development. The obvious benefits that you get by using VMs for 
development are:

* Isolation - don't bloat up your personal machine
* Consistency - no more "it works on my machine" issues
* Disposability - rebuild from scratch any time

Of course, working primarily in VMs can also be a hassle. Here's where 
vagrant comes in...

First of all, the VM is run headlessly in the background with your 
source files setup as a shared folder. This means you still work on the 
code locally on your host OS using your shell, your file manager, your 
editor. Just `ssh` into the VM whenever you need to to run your console, 
tests, and/or services. You can even hit these services from the host's 
browser.

Vagrant also automates the task of provisioning. This prevents both the 
reliance on a golden image as well as the hassle of manually managing 
the environment in the VM as your app changes.

You specify a "base box" which can be at any stage of built-out-ness. 
Usually, it's a vanilla server the same architecture and OS as your 
production boxes. You then specify chef recipes or puppet manifests to 
be run by vagrant to build out the box the rest of the way. For a long 
time now, Ops has understood configuration management, how to build out 
boxes from scratch in a reliable, repeatable way -- it's a solved 
problem. Vagrant lets us Devs leverage this too.

Beyond these benefits, vagrant does all this for a large variety of OS 
combinations and, so far, it's always Just Worked.

So If any of this sounds interesting, I encourage you to checkout 
[vagrantup.com][vagrant]; it's super easy to get started and there's a 
ton of articles, presentations, etc that can provide more details.

[vagrant]: http://vagrantup.com "title"
