---
title: Goodsong
date: 2009-12-05
tags: [shell]
---

If you're like me, (which you're probably not...) you enjoy
listening to your music with the great music playing daemon known
as [mpd][]. You also have your entire collection on shuffle.

[mpd]: http://mpd.wikia.com/wiki/Music_Player_Daemon_Wiki "mpd hompage"

Occasionally, I'll fall into a valley of bad music and end up
hitting next far too much to get to a good song. For this reason, I
wrote goodsong.

## What is it?

Essentially, you press one key command to say *the currently playing 
song is good*; then press a different key to say *play me a good song*.

Goodsong accomplishes exactly that. It creates a playlist file
which you can auto-magically add the currently playing song to with
the command `goodsong`. Subsequently, running `goodsong -p` will
play a random track from that same list.

Here's the `--help`:

```
usage: goodsong [ -p | -ls ]

options:
      -p,--play   play a random good song
      -ls,--list  print your list with music dir prepended

      none        note the currently playing song as good
```

## Installation

Goodsong is available in its current form in my [git repo][repo].

[repo]: https://github.com/pbrisbin/scripts/blob/pre-cleanout/goodsong

## Usage

Using goodsong is easy. You can always just run it from CLI, but I
find it's best when bound to keys. I'll leave the method for that
up to you; `xbindkeys` is a nice WM-agnostic way to bind some keys,
or you can use your a WM-specific configuration to do so.

Personally, I keep Alt-g as `goodsong` and Alt-Shift-g as
`goodsong -p`.

{{< well >}}
You're going to have to spend some time logging songs as "good"
before the `-p` option becomes useful.
{{< /well >}}

{{< well >}}
I recently received a patch from a reader for this script. It adds
a few features which I've happily merged in.

* Various methods are employed to try and determine exactly what 
  `mpd.conf` you're currently running with at the time
* The goodsong list is now a legitimate playlist file stored in your 
  `playlist_directory` as specified in `mpd.conf`
{{< /well >}}
