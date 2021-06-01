---
title: Scratchpad Everything
date: 2010-06-14
tags: [haskell, xmonad]
---

If you've read my recent [post](/posts/xmonad_scratchpad) on using a 
scratchpad in XMonad, and if you've actually implemented this in your 
own setup, you probably know how useful it is. For those that don't know 
what I'm talking about, you basically setup a simple keybinding that 
calls up a terminal (usually floated, but managed by its own specific 
manageHook) to be used briefly before being banished away by the same 
keybinding.

Recently, I found that you can apply this functionality to any
application you'd like.

## ScratchMixer

I have my music playing through MPD all the time. Occasionally,
I'll like to play some other media, a youtube video or what have
you. When I do this, I call up ossxmix, adjust down MPD, and adjust
up my browser (per application volumes are awesome by the way).

I realized that this was a perfect scratchpad scenario. I was
calling up this application for just a second, using it, then
sending it away. This simple activity was requiring that I M-p,
type ossxmix, hit enter, layout-shuffle, adjust volumes, then M-S-c
every single time. What was I thinking?

## XMonad.Util.NamedScratchpad

My last writeup used the contrib module `XMonad.Util.Scratchpad`
which, though it has a shorter name, simply provided wrapper
functions for the things I'm now using from
`XMonad.Util.NamedScratchpad`.

In the parent extension, things are much more transparent and free.
For me, this lead to a much cleaner config file too. I wish I had
been using things this way from the start.

So of course, we'll need to add
`import XMonad.Util.NamedScratchpad` to the top of our config
file.

{{< well >}}
Please refer back to my previous [post](/posts/xmonad_scratchpad/) for 
information regarding some boilerplate code. This writeup assumes you 
have a main-do block that calls out `myManageHook` and `myKeys` to be 
defined as separate functions. I also won't be going into hiding the NSP 
{{< /well >}}

## Scratchpads

The Named Scratchpad extension exposes a new data type that can be
used to represent a scratchpad. The following four things must be
specified to fully describe a scratchpad:

-   A String: the name to call it
-   A String: the command to launch
-   A Query Bool: The way to find the window once it's running
-   A ManageHook: The way to manage the window when we call it up

{{< well >}}
Those last two data types might sound scary, but they aren't. If
you think of the fact that most users define custom window
management in a list of `(Query Bool --> ManageHook)` and one
representation of this might be
`(className =? "Firefox" --> doFloat)` that should give you an idea
of the sorts of functions that you should use to fill those last
two slots for your scratchpads.
{{< /well >}}

The 
[haddocks](http://xmonad.org/xmonad-docs/xmonad-contrib/XMonad-Util-Scratchpad.html)
for this module talk about everything that's available, but here's a 
commented version of my declaration:

```haskell 
myScratchPads = [ NS "mixer"    spawnMixer findMixer manageMixer -- one scratchpad
                , NS "terminal" spawnTerm  findTerm  manageTerm  -- and a second
                ]

  where

    spawnMixer  = "ossxmix"                               -- launch my mixer
    findMixer   = className =? "Ossxmix"                  -- its window has a ClassName of "Ossxmix"
    manageMixer = customFloating $ W.RationalRect l t w h -- and I'd like it fixed using the geometry below:

      where

        h = 0.6       -- height, 60% 
        w = 0.6       -- width, 60% 
        t = (1 - h)/2 -- centered top/bottom
        l = (1 - w)/2 -- centered left/right

    spawnTerm  = myTerminal ++ " -name scratchpad"       -- launch my terminal
    findTerm   = resource  =? "scratchpad"               -- its window will be named "scratchpad" (see above)
    manageTerm = customFloating $ W.RationalRect l t w h -- and I'd like it fixed using the geometry below

      where

        -- reusing these variables is ok since they're confined to their own 
        -- where clauses 
        h = 0.1       -- height, 10% 
        w = 1         -- width, 100%
        t = 1 - h     -- bottom edge
        l = (1 - w)/2 -- centered left/right
```

So you can see I have a list containing two scratchpads. The
datatype syntax requires the "NS" plus the four things I've listed
above.

{{< well >}}
You'll notice I liberally use sub-functions via where clauses. You
can think of these as simple variables and if parenthesized and
placed directly where they're called out, they would work exactly
the same. I think this is clearer and it should be fairly obvious
how it works.
{{< /well >}}

The beauty of all this is that it's almost all that's needed. Each
scratchpad has a name which can be bound to a key; even better, the
whole scratchpad list will be managed with one simple addition to
your manageHook.

I inserted the following keybindings:

```haskell 
myKeys = [ ...
         , ...

         , ("M4-t"   , scratchTerm )
         , ("M4-S-m" , scratchMixer)

         , ...
         ] 

         where

           -- this simply means "find the scratchpad in myScratchPads that is 
           -- named terminal and launch it"
           scratchTerm  = namedScratchpadAction myScratchPads "terminal"
           scratchMixer = namedScratchpadAction myScratchPads "mixer"
```

{{< well >}}
I'm using 
[EZConfig](http://xmonad.org/xmonad-docs/xmonad-contrib/XMonad-Util-EZConfig.html) 
notation in my keybindings.
{{< /well >}}

And tacked the following onto the end of my managehook:

```haskell 
myManageHook = ([ -- whatever it might be...
                , ...
                , ...

                -- this manages the entire list of scratchpads 
                -- based on the query and hook listed for each
                ]) <+> namedScratchpadManageHook myScratchPads
```

That's it, a scratch terminal and a scratch mixer; but most
importantly, simple and transparent tools for adding any arbitrary
application (graphical or in-term) as a scratchpad application.

One final note about testing: As you're tweaking your queries and
hooks, be sure to call up the application, close it, then Mod-Q and
test your changes. If you've got a scratchpad still open from
before your last config change, it will still be using the old
ManageHook.
