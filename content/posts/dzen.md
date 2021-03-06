---
title: Dzen
date: 2012-04-29
tags: [linux]
---

Here's for a small change of pace...

I'd like to talk about a tool I've all but forgotten I'm even using (and 
that's a compliment to its stability and unobtrusiveness).

`dzen` is a great little application from the folks at [suckless][]. 
It's one of those *do one thing and do it well* types of tools. It's 
probably not useful at all for anyone with a bloated --ahem, excuse me-- 
featureful desktop environment or window manager (or both).

[suckless]: http://suckless.org/

In my case, I'm using just XMonad with its beautiful simplicity. This 
means, of course, that there's no out-of-the box... anything.

I've already [covered][] some of this from an XMonad perspective, so 
this post is more about dzen's general usefulness.

[covered]: /posts/xmonad_statusbars

## Volume

First up, a small visual notification when I adjust my volume:

![ossvol screenshot](/images/dzen/ossvol.png)

It fades in (implicitly thanks to [xcompmgr][]) for just a second when I 
adjust my volume and gives me that nice, unobtrusive indication of the 
volume level.

[xcompmgr]: https://wiki.archlinux.org/index.php/Xcompmgr

The actual volume adjustment can be done in many alsa or oss specific 
ways; for my implementation, just see the [script][] as it is live. 
Completely separate of that, however, we can just use dzen to show the 
notification:

[script]: https://github.com/pbrisbin/scripts/blob/master/ossvol

```bash 
level=$(get_it_from_alsa_or_oss)

# we use a fifo to buffer the repeated commands that are common with 
# volume adjustment
pipe='/tmp/volpipe'

# define some arguments passed to dzen to determine size and color.
dzen_args=( -tw 200 -h 25 -x 50 -y 50 -bg '#101010' )

# similarly for gdbar
gdbar_args=( -w 180 -h 7 -fg '#606060' -bg '#404040' )

# spawn dzen reading from the pipe (unless it's in mid-action already).
if [[ ! -e "$pipe" ]]; then
  mkfifo "$pipe"
  (dzen2 "${dzen_args[@]}" < "$pipe"; rm -f "$pipe") &
fi

# send the text to the fifo (and eventually to dzen). oss reports 
# something like "15.5" on a scale from 0 to 25 so we strip the decimals 
# and send gdbar an optional "upper limit" argument
(echo ${level/.*/} 25 | gdbar "${gdbar_args[@]}"; sleep 1) >> "$pipe"
```

Pretty easy, and about as light-weight as you can get.

## Status bar

Little known fact: you can use the ubiquitous [conky][] to feed a simple 
statusbar via dzen. This means you can also use dzen escapes in your 
`TEXT` block to do cool things:

![dzen screenshot](/images/dzen/dzen.png)

[conky]: http://conky.sourceforge.net/

My statusbar has the following "features"

* Shows CPU/Mem/Network
* The time, of course
* Shows "Now playing" information from MPD
* The music state (playing/paused) can be clicked to toggle it
* The track title, when clicked, will advance

And here's the `conkyrc` to achieve it:

```
background no
out_to_console yes
out_to_x no
override_utf8_locale yes
update_interval 1
total_run_times 0
mpd_host 192.168.0.5
mpd_port 6600

TEXT
[ ^ca(1, mpc toggle)${mpd_status}^ca()

  ${if_mpd_playing}- ${mpd_elapsed}/${mpd_length}$endif ]

  ^fg(\#909090)^ca(1, mpc next)${mpd_title}^ca()^fg() by

  ^fg(\#909090)${mpd_artist}^fg() from

  ^fg(\#909090)${mpd_album}^fg()

  Cpu: ^fg(\#909090)${cpu}%^fg()

  Mem: ^fg(\#909090)${memperc}%^fg()

  Net: ^fg(\#909090)${downspeedf eth0} / ${upspeedf eth0}^fg()

  ${time %a %b %d %H:%M}
```

{{< well >}}
Line breaks added for clarity.
{{< /well >}}

The most interesting part is the clickable areas: `^ca( ... )some 
text^ca()` defines an area of "some text" that can be clicked. The 
two arguments inside the first parens are are "which mouse button" and 
"what command to run". Pretty simple and damn convenient.

Then all you've got to do is call this from your startup script:

```bash 
$ conky -c ~/path/to/that | dzen2 -p -other -args
```

The `-p` option just means "persist" so the dzen will never close.

## Wrap-up

This was just two examples of some uses for a simple "pipe some text in 
and see it" GUI toolkit -- there are plenty [others][android-receiver].

[android-receiver]: /posts/android_receiver "Android Receiver"

This echoes one of the great things about open-source: something like 
this is so small, so simple, it could never have survived marketing 
meetings, planning sessions or cost-benefit analyses -- but here it is, 
and I find it oh-so-very-useful.
