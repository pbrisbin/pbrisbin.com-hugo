---
title: Wifi Pipe
date: 2009-12-05
tags: [linux]
---

So the other day when I was using wifi-select (awesome tool) to connect 
to a friends hot-spot, I realized, *"hey! This would be great as an 
openbox pipe menu!"*

I'm fairly decent in bash and I knew both netcfg and wifi-select were in 
bash so why not rewrite it that way?

## Wifi-Pipe

A simplified version of wifi-select which will scan for networks and 
populate an openbox right-click menu item with available networks. 
Displays security type and signal strength. Click on a network to 
connect via netcfg the same way wifi-select does it.

Zenity is used to ask for a password and notify of a bad connection. One 
can optionally remove the netcfg profile if the connection fails.

## Requirements

* netcfg
* zenity
* A `NOPASSWD` entry in sudoers for this script
* An entry in your `menu.xml`

The script now has its own github repo so it doesn't fall victim to 
bitrot. Please head [there][] for more installation details and a copy 
of the source.

[there]: https://github.com/pbrisbin/wifi-pipe
