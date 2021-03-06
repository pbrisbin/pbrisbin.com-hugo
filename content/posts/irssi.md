---
title: Irssi
date: 2010-03-20
tags: [linux]
---

Irssi is an IRC client. If that sentence made no sense, then read no further.
This post outlines my current irssi setup as I think it's quite nice and others
may wish to copy it.

{{< well >}}
**Note**: I've since moved to weechat. If anyone's interested, that config can
be found [here][weechat].
{{< /well >}}

[weechat]: https://github.com/pbrisbin/dotfiles/tree/v1.0/tag-weechat

## Screenshot

![Irssi Screenshot](/images/irssi/irssi.png)

## Config

For the longest time I didn't really touch `~/.irssi/config` except to set up
auto connections etc. Then I started using awl.pl (which I'll describe in the
scripts section). This meant I no longer had a use for one of the statusbars. So
for the sake of completeness, here is the change I made to get the statusbar
look you see in the screenshot:

    statusbar = {

        # <snip>

        default = {
          window = {

            # disable the default bar containing window list
            disabled = "yes";

            # window, root
            type = "window";
            # top, bottom
            placement = "bottom";
            # number
            position = "0";
            # active, inactive, always
            visible = "active";

            # list of items in statusbar in the display order
            items = {
              barstart = { priority = "100"; };
              time = { };
              user = { };
              window = { };
              window_empty = { };
              lag = { priority = "-1"; };
              more = { priority = "-1"; alignment = "right"; };
              barend = { priority = "100"; alignment = "right"; };
              active = { };
              act = { };
            };
          };

          # <snip>

          prompt = {
            type = "root";
            placement = "bottom";
            # we want to be at the bottom always
            position = "100";
            visible = "always";
            items = {
              barstart = { priority = "100"; };
              time = { };

              user = { }; # added my current nick here b/c it was the only useful
                          # item in the disabled bar

              prompt = { priority = "-1"; };
              prompt_empty = { priority = "-1"; };
              # treated specially, this is the real input line.
              input = { priority = "10"; };
            };

          };

          # <snip>

        };
      };

My full config (sans passwords) can be downloaded
[here](http://github.com/pbrisbin/irssi-config).

## Theme

The theme I currently use was originally *generane.theme*; I've gradually hacked
away at it until, at this point, it's entirely unlike that theme. I just call it
*pbrisbin.theme* and it can be found with the above dotfiles. It's a really grey
theme to go with my overall desktop. Messages from me are a bright-ish grey,
with messages to me as bright yellow. Actions (`/me` stuff) are magenta and
offset to the left which I really like.

## Bitlbee

Bitlbee is a killer app. It sets up a small-footprint IRC server on your local
machine, hooks into your various chat protocols (gchat, aim, facebook, twitter),
and let's you `/join` or `/query` them as if they were any other `#channel`.

This is great for someone like me who's gotten used to `/exec -o foo` and other
tricks that aren't possible in a normal chat client.

There are a lot of guides online for setting this up so I'm just going to list
out a few facts that it took me a minute to figure out or get used to:

* In the &bitlbee channel, any text *not* prefixed with a buddy's nick 
  is interpreted as a command to bitlebee itself.

* If you decide to chat with buddies by sending nick-prefixed messages 
  within the main &bitlbee channel, it's not a chatroom and they can't 
  see things you send to other nicks.

* Whether you decide to talk to a buddy via a nick-prefixed message or a 
  query, bitlbee remembers this and any future conversations initiated 
  by them will come in the same way by default.

## Scripts

And the best part, the scripts. All of these can be easily googled
for so I won't provide links; the versions on my box could even be
out of date anyway.

**cap_sasl.pl** - in an effort to streamline my dotfiles management, I 
was looking for ways to get plaintext passwords out of dotfiles. One 
such way is to use SASL for authentication to freenode. After getting 
the script, setup can be done via in-irssi commands as many existing 
how-tos outline. I got gummed up however because I fudged up the server 
name (freenode vs Freenode) when setting up sasl compared to when I had 
initially setup the connection...

This is why I prefer to do direct, in-file configuration. So, here are 
the portions of `.irssi/config` to support this:

```
servers = (
  {
    address = "irc.freenode.net";
    chatnet = "freenode";
    port = "6697";
    use_ssl = "yes";
    ssl_verify = "yes";
    ssl_capath = "/etc/ssl/certs/";
    autoconnect = "yes";
  },

  ...
```

And place a file as `~/.irssi/sasl.auth` with the following contents:

```
freenode	<primary nick>	<password>	DH-BLOWFISH
```

It's important that you use your primary nick or it won't work. For 
instance, I always talk as `brisbin` but that's just a secondary nick 
associated with my primary `brisbin33`, so I had to use `brisbin33` in 
the sasl setup.

**nm.pl** - this handles random/unique nick coloring and nick
alignment. Personally, I `/set neat_maxlength 13`.

**awl.pl** - the advanced window list (sometimes called
adv\_windowlist.pl). This gives that nice statusbar with the
channel names and numbers. Channels turn bright white when active
and magenta if I'm highlighted. Personally, I use
`/set awl_display_key "%w$N.$H$C$S"` and `awl_maxlines 1`.

**trackbar.pl** - this puts a dashed mark in the buffer at the last
point you viewed the conversation. I really like this script, it's
simple but affective. If you hop around between windows this is a
great little addition to your `.irssi/scripts/autorun`.

**screen\_away.pl** - thank you [rson](http://rsontech.net) for
turning me onto this. Once I started using irssi exclusively in
screen (as outlined [here](/posts/screen_tricks)) this script
really started coming in handy. It just auto-sets you as away when
you detach your screen session and brings you back when you
reattach. This means Ctrl-a d logs me off, and when I do reattach
I've got all my messages waiting for me right there in window 1.

**queryresume.pl** - now that I'm using bitlbee as my main IM client, 
I'm spending a lot of time in queries. This script gives you a little 
bit of context by printing the last few lines of your most recent query 
with this person that you've just started a new query with.

**hilightwin.pl** - this script captures any text that matches your 
`/hilight` rules, whether it's nick or keyword-based. Anything you've 
set up as a hilight will be captured in a dedicated window. Couple this 
with a smart layout where your hilightwin is dedicated to the top 8 
lines of your client, and you can always see who's talking at you, no 
matter what you're doing. Any google search for this script will not 
only give you the source, but also the commands required to setup the 
smart layout to go along with it.

**link_titles.pl** - this is a script that I recently wrote as a learning
exercise in perl. It watches the conversation for urls. When it finds one, it
visits that page and prints the `title` element back to the window where the
link was sent. Most actual channels I'm in will have a bot that does this, but I
wanted to print titles for links sent to me in a query via gchat or aim. The
source for this is on my [github](https://github.com/pbrisbin/irssi-scripts),
hopefully more scripts will show up there soon.
