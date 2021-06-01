---
title: Chruby
date: 2013-04-07
tags: [ruby]
---

Once I start my new job at [thoughtbot][], I'll be working on a variety 
of ruby and rails projects at the same time. This, combined with the 
current 2.0 transition, means I once again need a ruby version 
management tool.

[thoughtbot]: http://thoughtbot.com

[Chruby][] is the third (by my count) "new hotness" when it comes to 
these python-inspired virtualenv clones. First there was [rvm][] which 
has a ton of features, then came [rbenv][] which aimed to be simpler, 
finally we have chruby which is billed as the simplest of them all. So 
far, I'm a big fan.

[chruby]: http://github.com/postmodern/chruby
[rvm]:    http://github.com/wayneeseguin/rvm
[rbenv]:  http://github.com/sstephenson/rbenv

{{< well >}}
For detailed instructions and usage please see the `README` files in the 
previously linked project pages. This post might gloss over some details 
and focuses more on my opinion of the tools than their usage. For Arch 
users, there are AUR PKGBUILDs for all of these.
{{< /well >}}

## Choices

Last time I required this feature, rbenv was just coming onto the scene, 
so I went with rvm. It is by far the most complex of these tools, and 
that is a downside itself. Overwriting `cd` (to allow auto-switching) is 
a concern for some people. The fact that it both installs and manages 
versions strikes others as a breach of [Unix][].

[unix]: http://en.wikipedia.org/wiki/Unix_philosophy#Mike_Gancarz:_The_UNIX_Philosophy

One feature commonly touted as the reason to use rvm is its gemsets 
which isolate sets of gems into groups and thus prevent gem-hell. Now 
that bundler is ubiquitous, this problem no longer exists.

Aside from rvm, the other major choices are rbenv and chruby. Looking at 
the rbenv project page, it still seems to do a number of things I don't 
need or want. I'm also not a fan of it introducing a bunch of shims.

At its core, all such a manager needs to do is modify some environment 
variables so that the correct binary and set of libraries are loaded. 
Coincidentally, that's about all chruby does.

## Chruby

Paraphrasing from the project page, changing rubies via `chruby` will:

1. Update `$PATH` so the correct ruby and any gem executables are 
   directly available.
2. Set a proper `$GEM_HOME` and `$GEM_PATH` so any gem related commands 
   and tools (including bundler) will Just Work.
3. Set some other ruby-related environment variables.
4. Call `hash -r` for you (required when mucking with `$PATH`).

No shims, no crazy options or features bloating up the script which 
itself weighs in at less than 90 lines of very simple and readable 
shell.

If you choose, chruby can also do automatic switching. To opt in, you 
just have to source an additional (and equally simple) script. Once 
enabled, you will automatically change rubies when you enter a directory 
containing a `.ruby-version` file. This is done cleanly via a pre-prompt 
command and not by hijacking `cd`.

{{< well >}}
When auto-switching is enabled, be sure to define a "default" by 
dropping a `.ruby-version` in `$HOME` too.
{{< /well >}}

Here are the entries in my `~/.zshenv` (the same should work in bash):

```bash 
if [[ -e /usr/share/chruby ]]; then
  source /usr/share/chruby/chruby.sh
  source /usr/share/chruby/auto.sh
  chruby $(cat ~/.ruby-version)
fi
```

{{< well >}}
The AUR PKGBUILD installs into `/usr/share` while the chruby README 
prescribes `/usr/local/share`. This may be a packaging bug that will 
eventually be fixed so be sure to verify and use the appropriate paths 
for your install.
{{< /well >}}

So far, I'm a huge fan. The tool does what it advertises exactly and 
simply. The small feature-set is also exactly and only the features I 
need. As a bonus, setting the `GEM_` variables is something I always 
seemed to need to do manually anyway, so it's nice to no longer need 
that.

## Ruby-build

Since chruby is just a "changer" you do need to install rubies via some 
other tool. [Ruby-build][] makes that super easy:

[ruby-build]: https://github.com/sstephenson/ruby-build

```
$ ruby-build 1.9.3-p392 ~/.rubies/ruby-1.9.3-p392
$ ruby-build 2.0.0-p0 ~/.rubies/ruby-2.0.0-p0
```

{{< well >}}
Chruby will look for rubies installed in one of two places by default: 
`/opt/rubies/` or `~/.rubies/`. I prefer the latter.
{{< /well >}}

Since ruby-build is actually a sub-tool of rbenv, it's quite spartan. 
You're required to type the desired version exactly (as read from 
`ruby-build --definitions`) and you need to give the full installation 
path, even though it could be determined easily by convention. `rbenv 
install` owns those niceties, apparently.

{{< well >}}
After this post was written, the author of chruby actually released a 
ruby-build competitor called [ruby-install][]. It's feature-set is very 
much the same and it allows fuzzy commands like `ruby-install ruby 1.9`. 
I very much recommend it.
{{< /well >}}

[ruby-install]: https://github.com/postmodern/ruby-install

## One last bit...

Some time ago, while still using both [oh-my-zsh][] and rvm, I noticed 
that most of the prompts used yet-another rvm feature to read the 
currently active ruby and insert it into the prompt.

[oh-my-zsh]: https://github.com/robbyrussell/oh-my-zsh

This seems a bit odd for a tool to provide this feature. There are also 
a great many `if` statements out there doing something different for rvm 
or rbenv. Will they all add a clause for chruby now?

Well, in a bout of insane cleverness, I found the following non-obvious 
way to get the currently active ruby version:

```
$ ruby --version
```

If you'd like to use this in your prompt, feel free to bogart from 
[mine][zsh-prompt].

[zsh-prompt]: https://github.com/pbrisbin/zsh-config/blob/master/functions/prompt_minimal_setup#L17
