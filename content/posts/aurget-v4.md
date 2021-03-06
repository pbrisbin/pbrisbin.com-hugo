---
title: Aurget v4
date: 2013-04-23
tags: [arch]
---

Aurget was one of the first programs I ever wrote. It's seen decent 
adoption as far as AUR helpers go and it's gradually increased its 
feature set over the past number of years.

The codebase had gotten a bit krufty and hard to follow. I decided to 
refactor to more isolated functions which didn't rely on so many global 
variables. This directed refactor has improved things greatly: pretty 
much any function can be reasoned about in isolation and the logic flows 
more understandably during program execution.

Unintentionally, this push to simplify and clarify actually resulted in 
a number of user-facing and developer-facing improvements. Go figure.

{{< well >}}
Listen -- I get it. I'm a minimalist too. Why do we need 400 lines to do 
essentially [this][]?

Funny thing is, no matter how hard I tried to chip aurget down closer to 
those essential `curl | tar | makepkg` parts, I would inevitably end up 
with sprawling, spaghetti code after the very first feature beyond what 
the above script provides. Available upgrades? Bloat. Dependency 
resolution? Bloat.

So after trying and giving up a number of times, I've decided aurget is 
actually a fairly simple and straight forward implementation of the 
features it currently provides, despite being so big.

In my opinion, it can't get much simpler without dropping features, and 
I actually like the features. Oh well, back to the post...
{{< /well >}}

[this]: https://github.com/pbrisbin/scripts/blob/master/aur

## Stupid Networking

In so many places, aurget would hit the RPC in a per-package way. 
Refactoring the networking made it obvious when I could use the 
multiinfo endpoint to query for many packages at once. This made many 
actions **way** faster.

Speaking of networking, it's now consolidated into a single `get` 
function. This means I can more easily play with `curl` options, error 
handling or even caching.

## Pass the Buck

Package installation is now handled by simply passing `--install` to 
makepkg. This has a number of positive consequences: Goodbye `sudo`, and 
so long to any configuration around the pacman command. Split packages 
are also handled predictably and you'll never have a successful build 
error with "package not found".

## Moar Options

Aurget will now pass any unknown options directly to `makepkg` (with a 
warning). This means anything `makepkg` supports, `aurget` supports too.

* Run as root
* Install a subset of a split package
* Package signing

Etc...

## Zomg DEBUG

One of the most frustrating aspects of working with aurget as its 
maintainer was troubleshooting. It was always both difficult and 
annoying to figure out what aurget was doing.

With the code now refactored such that the runtime behavior was more 
linear and understandable, a useful `--debug` flag could also be added.

I love this **so much**:

![ossvol screenshot](/images/aurget-v4/aurget-v4-debug.png)

Bring on the bugs!

Seriously though, there may be some. I changed a whole lot and didn't 
test exhaustively... Enjoy!
