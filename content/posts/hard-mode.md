---
title: Hard Mode
date: 2013-03-16
tags: [vim]
---

Recently, while watching Corey Haines and Aaron Patterson 
[pair-program][peep], I heard Mr. Haines mention vim's "hard mode". 
Apparently, this is when you disable the motion commands h, j, k, and l.

[peep]: https://peepcode.com/products/play-by-play-aaroncorey

It's absurd how great this exercise is for increasing your knowledge of 
vim. There are so many *better* ways to do *everything*. Just like 
complete novices might map the arrow keys to `Nop` to force learning 
`hjkl`, mapping the `hjkl` keys to `Nop` forces you to learn all these 
other ways to move around and edit parts of the file.

The real philosophical shift is thinking in *Text Objects* rather than 
Lines and Characters. Words are things, sentences are things, method 
definitions are things, and these can all be manipulated or navigated 
through as such.

While you probably can't fully internalize this concept without going 
through the [exercise][hardmode] yourself, I would like to share a few 
of the very first "better ways" I've been finding while restricted in 
this way.

[hardmode]: https://github.com/wikitopian/hardmode

## Just Search

Imagine my cursor is a ways down the document, and I need to change the 
above header in some way. I'm staring at "Search", I know I want my 
cursor there. I used to just tap `k` or maybe a few `10k`s with a `j` or 
two. What was I thinking?

```
?Se
```

And I'm there. In this case, the capital "S" made this word rare enough 
that I didn't have to type very much of it. Recognizing the relative 
frequency of words or characters can be a useful skill for quicker 
navigation. Drew Neil, author of [practical vim][practical], calls this 
"Thinking like a scrabble player".

[practical]: http://pragprog.com/book/dnvim/practical-vim

## Use the Ex, Luke
Another thing I didn't realize I do a lot is move to some far away line 
to copy it, only to come right back to paste it. Really? I'm going to 
type a bunch of `j`s only to then type the exact same number of `k`s?

You could use search to get to the far away line then double-backtick to 
jump back, or you could do this:

```
:2,7co .
```

This takes lines 2 to 7 and copies them to here. Not only is this less 
key-strokes (a number which grows proportional to the distance between 
here and there), but I'd argue it also keeps your focus better.

You can actually cut out a lot of unnecessary motion using commands like 
this:

```
:20   " go to line 20
:20d  " delete line 20
:2,7d " delete lines 2 through 7
```

In any of these commands `.` can be used to mean the current line. If 
you really get frustrated, you could use `:.+1` and `:.-1` to move like 
`j` and `k` -- but I wouldn't recommend it.

## Finding Character

It's times like these that I try to find a good first concept. Something 
that's going to be useful enough to get me further along the 
habit-building path, but simple enough that I don't have to remember too 
much.

First, know that `0` puts you at the start of the line. This gives you a 
common reference to move from so you only have to think in one direction 
(for now). Second, know that `f` and `t` go to a letter (so `fa` to go 
to the next "a" in the line). The difference is `t` goes `t`ill the 
character, stopping with the cursor just before it and `f` puts the 
cursor right on top. You can then use `;` to repeat the last search, 
moving a-by-a along the line.

Once you've gotten the hang of this, the capital versions, `F` and `T` 
do the same thing but backwards. `,` is the key to repeat the last 
backwards search, but so many people (including me) map that to `Leader` 
or `LocalLeader` that it's difficult to rely on. I haven't found a good 
solution to this, since the only other convention I know of is the 
default `\` which I can rarely type consistently.

{{< well >}}
There's a bit of stategy here. It's true of most motions, but it's most 
recognizable with `f`. You have two choices in approach: pick the letter 
that you want to be at (no matter what letter it is) and use `;` to 
repeat the last `f` or `t` until it gets you there (regardless of how 
many key strokes that is), or you can choose a letter that appears first 
in the line (knowing that it will only take one stroke to get there) but 
which only gets you *near* your goal. These are the two extremes, 
finding the best middle ground (lowest overall keystrokes) for any given 
scenario is something worth mastering.
{{< /well >}}

## Word-wise

In addition to finding by character we can start to think in words. 
Again, we're making it easy by always starting from `0`. Given that, 
just use `w` to move `w`ord by word with the cursor on the front of each 
word or `e` to move word by word but with the cursor on the `e`nd of 
each word. Eventually, I'll attempt to internalize the same commands in 
the other direction: `b` and `ge`.

{{< well >}}
All of these have capital versions (`W`, `B`, `E`, `gE`) which have the 
same behavior but work on `WORDS` not `word`s.

The exact rules about `word`s vs `WORD`s aren't worth memorizing. 
`WORD`s are basically just a higher level of abstraction. For example, 
`<foo-bar>` is 5 `word`s but it's only one `WORD`.
{{< /well >}}

## Conclusion

So far, I've gotten myself to consistently use a number of new vim 
tricks:

1. Use search to get where you want
2. Use Ex commands to manipulate text not near the cursor
3. Move by word, not by character

There's still plenty to learn, but I've found that just these few simple 
ideas make me effective enough that I'm sticking with it and not just 
giving up in frustration.
