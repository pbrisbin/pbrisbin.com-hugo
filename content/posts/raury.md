---
title: Raury
date: 2012-08-30
tags: [arch, ruby]
---

*tl;dr: it's just like aurget but more stable and faster*

Developing aurget was getting cumbersome. Whenever something went wrong, 
it was very difficult to track down or figure out. The lack of standard 
tools for things like uri escaping or json parsing was getting a bit 
annoying, and the structure of the code just annoyed me. There was also 
a lack of confidence when changes were made, I could only haphazardly 
test a handful of scenarios so I was never sure if I'd introduced a 
regression.

I decided to write [raury][] to be exactly as featureful as aurget, but 
different in the following ways:

[raury]: https://github.com/pbrisbin/raury

* Solid test coverage

![raury coverage](/images/raury/coverage.png)

* Useful debug output

![raury debug output](/images/raury/debug.png)

* Clean code

![raury code](/images/raury/code.png)

I think I've managed to hit on all of these with a happy side-effect 
too: it's really fast. It takes less than a few seconds to churn through 
a complex tree of recursive dependencies. The same operation via aurget 
takes minutes.

## Interested?

So anyway, if you're interested in trying it out, I'd love for some beta 
testers.

Assuming you've got a working ruby environment (and the bundler gem), 
you can do the following to quickly play with raury:

```
$ git clone https://github.com/pbrisbin/raury && cd ./raury
$ bundle
$ bundle exec bin/raury --help
```

If you like it, you can easily install it for real:

```
$ rake install
$ raury --help
```

There's also a simple script which just automates this clone-bundle-rake 
process for you:

```
$ curl https://github.com/pbrisbin/raury/raw/master/install.sh | bash
```

{{< well >}}
Also, tlvince was kind enough to create a PKGBUILD and even maintain an 
AUR [package][raury-git] for raury. Check that out if it's your 
preferred way to install.
{{< /well >}}

[raury-git]: https://aur.archlinux.org/packages.php?ID=63129
