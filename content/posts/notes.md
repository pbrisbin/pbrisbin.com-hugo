---
title: Notes
date: 2011-03-26
tags: [linux]
---

For me, any sort of general purpose note taking and/or keeping solution 
needs to meet only a few requirements:

1. Noting something has to be quick and easy (in a terminal and scriptable)
2. Notes should be available from anywhere... *tothe[cloud][]!*
3. Notes should be searchable

Now, just to clarify -- I'm not talking about classroom notes, those 
things go in note-books. I'm talking about short little blurbs of 
information I would like to keep and reference at a later time. 

Though, I suppose this *could* work for classroom notes too...

I'm also not talking about reminders, those are the stuff of calendars, 
not note-keeping apps.

So what's my solution? What else, Gmail!

## Gmail

Setting up gmail as a note keeper/searcher is simple. A note is an email 
from me with the prefix "Note - " in the subject line. Therefore, it's 
easy to setup a label and a filter to funnel note-mails into a defined 
folder:

    From:    me@whatever.com
    Subject: ^Note - 

I also add "Skip inbox" and "Mark as read" as part of the rule.

{{< well >}}
I know the gmail filters support some level of regex and/or globbing, 
but I don't know where it ends. I'm hoping that the `^` anchor is 
supported but I'm not positive.
{{< /well >}}

Requirements 2 and 3 done.

## Mutt

So if taking a note is done by just sending an email of a particular 
consistent format, then it's easy for me to achieve requirement 1 since 
I use that awesome terminal mail client mutt.

A short bash function gives us uber-simple note taking abilities by 
handling the boilerplate of a note-mail:

```bash 
noteit() {
  _have mutt || return 1 # see my dotfiles re: _have

  local subject="$*" message

  [[ -z "$subject" ]] && { echo 'no subject.'; return 1; }

  echo -e "^D to save, ^C to cancel\nNote:\n"

  message="$(cat)"

  [[ -z "$message" ]] && { echo 'no message.'; return 1; }

  # send message
  echo "$message" | mutt -s "Note - $subject" -- pbrisbin@gmail.com

  echo -e "\nnoted.\n"
}
```

{{< well >}}
You could probably also streamline note taking by leveraging mutt's -H 
option. I'll leave reading that man page snippet as an exercise to the 
reader.
{{< /well >}}

And here's how that might work out in the day-to-day:

    //blue/0/~/ noteit test note
    ^D to save, ^C to cancel
    Note:

    This is a test note.

    < I pressed ^D here >
    noted.

    //blue/0/~/

{{< well >}}
You could also use `sendmail`, `mailx`, `msmtp` or whatever other <abbr 
title="command line interface">CLI</abbr> mail solution you want for 
this.
{{< /well >}}

And there it is, ready to be indexed by the almighty google:

![Mutt notes shot](/images/notes/mutt-notes.png)

With a few mutt macros, I think this could get pretty featureful without 
a lot of code.

Let me know in the comments if there are any other simple or 
out-of-the-box note-keeping solutions you know of.

Oh, and before anyone mentions it -- no, you can't take notes without 
internet when you're using this approach. I'm ok with that, I understand 
if you're not.

[cloud]: http://farm6.static.flickr.com/5001/5191055713_31af0bc3e7.jpg "lol"
