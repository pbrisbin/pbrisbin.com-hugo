---
title: HTPC
date: 2010-05-01
tags: [linux]
---

I've recently finished work on an HTPC. The goal was to run a media 
center WM on a box that looked appropriate in my cabinet by my TV using 
a remote. That much I've done; all that's left is tweaking the remote 
functions and adding to the collection.

## Hardware

The first thing I got was the case; I wanted one with a built in remote 
and a low enough profile to fit in my TV cabinet and not look out of 
place.

Enter [Lian Li's PC-C39][lian]. Let me say, it's a great case. It's 
small, quiet, and looks great. One problem, the remote is garbage.

[lian]: http://www.newegg.com/Product/Product.aspx?Item=N82E16811112230

It doesn't work more than 2 feet away from the sensor. The remote is RF 
(another flaw IMO) and the sensor is actually over-shielded by the case 
itself. Solution? Slide open the top of the case (even just an inch), 
your range will increase tenfold. I did this for a while but wanted 
something better -- more on that later; anyone reading this should buy 
the PC-C37B which is the same case but sans the trash remote (and $50 
bucks).

Next, I stopped in at MicroCenter to pick up the internal components. I 
knew I wanted to spend five to six hundred bucks and get a decently 
powered machine; one that could keep up with whatever HD content I 
wanted to run without getting too hot.

Here's what I ended up with:

- Intel DP55WB mATX 129.99
- ASUS GF210 47.99
- Intel Core i5 650 139.99
- OCZ 4GB DDR3 1600 CL8 119.99
- OCZ 600W Stealth 69.99
- WD 1TB SATA 99.99
- LG 22x Burner 29.99
- Total **639.93**

After the usual mail-in-rebates, It'll be just over $550. You could 
definitely achieve a great system for less, but I wanted something more 
high-end (and I had just gotten my tax return), so I probably spent a 
little more than I had to.

So now that I've got a fully functioning box, it's time to fix my remote 
situation.

Enter [Logitech's Harmony 300][harmony]. I originally bought this 
thinking it was primarily a PC Media Center remote and would come with 
its own USB IR receiver. It did not. I was pissed.

[harmony]: http://www.newegg.com/Product/Product.aspx?Item=N82E16880111033

In the end, I'm really glad I made that mistake because the remote's 
awesome. You configure it by plugging it into a computer and using an 
in-browser control panel (luckily it's mac+firefox compatible), just add 
devices by Manufacturer number, and that's it.

To get it working with the computer was a bit more involved, but not 
much.

First, I had to get my own USB IR Receiver. Luckily, amazon had a [Dell 
RC6 receiver][dell] for like $18 bucks, sold. Then it was just a matter 
of adding its MFR\# to the harmony setup and starting lirc.

[dell]: http://www.amazon.com/Dell-RC6-Receiver-Microsoft-receivers/dp/B0031YBF9M/ref=sr_1_3?ie=UTF8&s=electronics&qid=1272722402&sr=8-3

If you're on Arch, it's like this:

    pacman -S lirc
    cp /usr/share/lirc/remotes/mceusb/lircd.conf.mceusb /etc/lirc/lircd.conf
    /etc/rc.d/lircd start

You can test it by typing `irw` and pressing some buttons.

You'll want to add `lirc_mceusb2` to `MODULES` and `lircd` to
`DAEMONS` in `/etc/rc.conf`.

{{< well >}}
If you find on reboot that your remote's not working, check if 
`/dev/lirc0` exists (it needs to); if this happens, try a different USB 
port, that solved it for me
{{< /well >}}

Now I've got just one remote that runs my whole living room. The 
girlfriend was pleased. There was much rejoicing.

## Software

I went with [XBMC](http://xbmc.org). Once installed, I set up an 
autologin by editing `/etc/inittab` (assuming xbmc is your default 
username):

```
## Only one of the following two lines can be uncommented!
# Boot to console
#id:3:initdefault:
# Boot to X11
id:5:initdefault:

# snip...

x:5:respawn:/bin/su xbmc -l -c "/bin/bash --login -c startx >/dev/null 2>&1"
```

And then adding the following to that user's `~/.xinitrc`:

```
exec /usr/bin/ck-launch-session /usr/bin/dbus-launch --exit-with-session /usr/bin/xbmc --standalone -fs
```

{{< well >}}
Most of the above is out of date now. I defer to the [Arch wiki][wiki] for details on
setting up XBMC.
{{< /well >}}

[wiki]: https://wiki.archlinux.org/index.php/Xbmc

I share my media from the main desktop PC using samba, so I just added 
the shares in XBMC.

Once added, XBMC scans your sources using some filename regexps that 
caught pretty much everything I threw at it. It downloaded plot 
summaries and fanart for all my movies and TV shows, and it of course 
uses your music collection's tags (which I'm a bit OCD about anyway).

The result is an instantly full and beautiful library. Here are some 
screenshots:

![HTPC Shot](/images/htpc/htpc-0.bmp)
![HTPC Shot](/images/htpc/htpc-1.bmp)
![HTPC Shot](/images/htpc/htpc-2.bmp)
![HTPC Shot](/images/htpc/htpc-4.bmp)

## Remote configuration

XBMC found and used a hotplugged keyboard, the case's built-in RF 
remote, and my lirc controlled mceusb remote all without issue right out 
of the box using default button mappings. I was impressed.

If you'd like to customize your remote behavior, there are two files 
involved: `~/.xbmc/userdata/Lircmap.xml` and 
`~/.xbmc/userdata/keymaps/remote.xml`. Defaults can be found in 
`/opt/xbmc/system` on an Arch install; just copy them and start editing.

`Lircmap.xml` will translate the device/button (as reported by `irw`) to 
an XBMC button string. Through this file, you can make it so that `... 
OK mceusb` will register as "select". Then, in `remote.xml` you can 
actually map `select` to an XBMC action, like "Select".

{{< well >}}
It's all explained
[here](http://wiki.xbmc.org/index.php?title=Lirc_and_Lircmap.xml)
and [here](http://wiki.xbmc.org/?title=Keymap.xml#Remote_Section).
{{< /well >}}

The last little issue I noticed was that after playing a DVD, I couldn't 
eject. This was fixed by adding the following line to the file 
`/etc/sysctrl.conf`:

    sys.dev.cdrom.lock = 0

A reboot is required for the change to take effect.

{{< well >}}
With the update to the 2.6.34 kernel, alsa now has support for audio 
over hdmi with my chipset (Asus/Nvidia GF210).

It wasn't exactly trivial to get it working though. Basically it took 
some trial and error to figure out that the audio out I needed was card 
1 device 7, so `plughw:1,7`.

Sadly, specifying this plughw as a custom output device in XBMC's audio 
setup meant no dmix, which meant no crossfading (two sounds at once).

Thanks to
[Themaister](http://bbs.archlinux.org/viewtopic.php?pid=789152#p789152)
on the arch forums though, I actually got around this quite quickly.

Save the following as `/etc/asound.conf`:

    pcm.dmixer {
      type dmix
      ipc_key 2048
      slave {
        pcm "hw:1,7"
        period_size 512
        buffer_size 4096
        rate 48000
        format S16_LE
      }
      bindings {
        0 0
        1 1
      }
    }

    pcm.!default {
      type plug
      slave.pcm dmixer
    }

    pcm:iec958 {
      type plug
      slave.pcm dmixer
    }

Reboot.

In the XBMC audio setup, specify `default` as the output device and 
`iec958` as the passthrough device.

That's it!
{{< /well >}}
