---
title: Mairix
date: 2011-07-03
tags: [mutt]
---

Mairix is a nice little utility for indexing and searching your emails. 
Its smooth integration with mutt is also a plus.

I used to use native mutt search, but it's pretty slow. So far, mairix 
is giving me a good approximation of the google-powered search available 
in the web interface and it's damn fast.

As I go through this setup, keep in mind the example config files are 
designed to work with my overall mutt setup; one which is described in 
two other posts [here][1] and [here][2].

[1]: /posts/mutt_gmail_offlineimap "Mutt + Gmail + Offlineimap"
[2]: /posts/two_accounts_in_mutt   "Using Two IMAP Accounts in Mutt"

{{< well >}}
If you need a little context, checkout my [mutt-config repo][repo] which 
has a fully functioning `~/.mutt`, example files for the other apps 
involved (offlineimap, msmtprc, and now mairix), and any scripts the 
setup needs.
{{< /well >}}

[repo]: https://github.com/pbrisbin/mutt-config "mutt-config on github"

## Mairix

First, of course, install mairix:

```
pacman -S mairix
```

Then, setup a `~/.mairixrc` which defines where your mails are and their 
type as well as where to store the results and index. Here's an example:

```bash 
# where you keep your mail
base=/home/<you>/Mail

# colon separated list of maildirs to index.
#
# I have two accounts each in their own subfolder. the '...' means there 
# are subdirectories to search as well; it's like saying GMail/* and 
# GMX/*
maildir=GMail...:GMX...

# I omit gmail's archive folder so as to pevent duplicate hits
omit=GMail/all_mail

# search results will be copied to base/<this folder> for viewing in 
# mutt
mfolder=mfolder

# and the path to the index itself
database=/home/<you>/Mail/.mairix_database
```

With that in place, run `mairix` once to build the initial index. This 
first run will be slower but in my tests, subsequent rebuilds were almost 
instant.

{{< well >}}
In situations like these, I'll usually add a verbose flag so I can be 
sure things are working as expected.
{{< /well >}}

At this point, you could actually do some searching right from the 
commandline:

```
mairix some search term # search and populate mfolder
mutt -f mfolder         # open it in mutt
```

This wasn't the usage I was after however, I'm typically already in mutt 
when I want to search my mails.

## Mutt

My original script for this purpose was pretty simple. It prompted for 
the search term and ran it. The problem was you then needed a separate 
keybind to actually view the results.

Thankfully, Scott commented and provided a more advanced script which 
got around this issue. Many thanks to Scott and whomever wrote the 
script in the first place.

This version does some manual tty trickery to build its own prompt, read 
your input, execute the search and open the results. All from just one 
keybind.

I merged the two scripts together into what you see below. The main 
changes from Scott's version are the following:

1. I kept my clear, purge, search method rather than relying on cron to 
   keep the index up to date.
2. I removed the append-search functionality; not my use-case.
3. I removed the `<return>` from the ^G trap; it was getting executed by 
   mutt and opening the first message in the inbox after a cancelled 
   search.
4. I fixed it so that backspace works properly in the prompt.

So, here it is:

```bash 
#!/bin/bash

read_from_config() {
  local key="$1" config="$HOME/.mairixrc"

  sed '/^'"$key"'=\([^ ]*\) *.*$/!d; s//\1/g' "$config"
}

read -r base    < <(read_from_config 'base')
read -r mfolder < <(read_from_config 'mfolder')

# prevent rm / further down...
[[ -z "$base$mfolder" ]] && exit 1

searchdir="$base/$mfolder"

set -f                          # disable globbing.
exec < /dev/tty 3>&1 > /dev/tty # restore stdin/stdout to the terminal,
                                # fd 3 goes to mutt's backticks.
saved_tty_settings=$(stty -g)   # save tty settings before modifying
                                # them

# trap <Ctrl-G> to cancel search
trap '
  printf "\r"; tput ed; tput rc
  printf "/" >&3
  stty "$saved_tty_settings"
  exit
' INT TERM

# put the terminal in cooked mode. Set eof to <return> so that pressing
# <return> doesn't move the cursor to the next line. Disable <Ctrl-Z>
stty icanon echo -ctlecho crterase eof '^M' intr '^G' susp ''

set $(stty size) # retrieve the size of the screen
tput sc          # save cursor position
tput cup "$1" 0  # go to last line of the screen
tput ed          # clear and write prompt
tput sgr0
printf 'Mairix search for: '

# read from the terminal. We can't use "read" because, there won't be
# any NL in the input as <return> is eof.
search=$(dd count=1 2>/dev/null)

# clear the folder and execute a fresh search
( rm -rf "$searchdir"
  mairix -p
  mairix $search
) &>/dev/null

# fix the terminal
printf '\r'; tput ed; tput rc
stty "$saved_tty_settings"

# to be executed by mutt when we return
printf "<change-folder-readonly>=$mfolder<return>" >&3
```

A non-trivial macro provides the interface to the script. It sets a 
variable called `my_cmd` to the output of the script, which should be 
the actual `change-folder` command, then executes it.

```
macro generic ,s "<enter-command>set my_cmd = \`$HOME/.mutt/msearch\`<return><enter-command>push \$my_cmd<return>" "search messages"
```

{{< well >}}
I've gotten used to "comma-keybinds" from setting that as my localleader 
in vim. It's nice because it very rarely conflicts with anything 
existing and it's quite fast to type.
{{< /well >}}

One downside which I've been unable to fix (and believe me, I've tried!) 
is that if you press ^G to cancel a search but you've typed a few 
letters into the prompt, mutt will read those letters as commands (via 
the `push`) and execute them.

The only thing I could do is prefix those characters with something. 
I've decided to use `/`. That makes mutt see it as a normal search which 
you can execute or ^G again to cancel. Annoying, but better than mutt 
flailing around executing rando commands...

## Power search

I haven't had the time yet to learn all the tricks, but here are some of 
the more useful-looking searches from `man mairix`:

```
Useful searches

   t:word                             Match word in the To: header.

   c:word                             Match word in the Cc: header.

   f:word                             Match word in the From: header.

   s:word                             Match word in the Subject: header.

   m:word                             Match word in the Message-ID: 

                                      header.

   b:word                             Match word in the message body 
                                      (text or html!)

   d:[start-datespec]-[end-datespec]  Match messages with Date: headers 
                                      lying in the specific range.

Multiple body parts may be grouped together, if a match in any of them 
is sought.

   tc:word  Match word in either the To: or Cc: headers (or both).

   bs:word  Match word in either the Subject: header or the message body 
            (or both).

   The a: search pattern is an abbreviation for tcf:; i.e. match the 
   word in the To:, Cc: or From: headers.  ("a" stands for "address" in 
   this case.)

The "word" argument to the search strings can take various forms.

   ~word        Match messages not containing the word.

   word1,word2  This matches if both the words are matched in the 
                specified message part.

   word1/word2  This matches if either of the words are matched in the 
                specified message part.

   substring=   Match any word containing substring as a substring

   substring=N  Match any word containing substring, allowing up to N 
                errors in the match.

   ^substring=  Match any word containing substring as a substring, with 
                the requirement that substring occurs at the beginning 
                of the matched word.
```

Happy searching!
