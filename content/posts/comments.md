---
title: Comments
date: 2011-07-08
tags: [haskell, self, yesod]
---

**Note**: this page describes a custom system for commenting once present on
this site. As you can see, it no longer is. Please provide any comments via
twitter or email. Thanks.

Recently I decided I no longer like disqus. It was a great hands-off 
commenting system, but it had its downsides. I decided to make a change.

I have my own commenting [system][] which I wrote as a yesod module. The 
initial reason I didn't go with this when I first moved to yesod (and 
lost my existing homebrew php commenting system) was because it didn't 
support authentication. I knew that an unauthenticated "enter comments" 
box had like a 100 to 1 ratio of spam to real comments.

[system]: https://github.com/pbrisbin/yesod-comments

When I put up [rentersreality][], I took the time to build 
authentication into my comments system by natively supporting 
`YesodAuth` so that I could use my module there. With that in place, the 
only thing keeping me back was losing my existing comments (more on that 
later).

[rentersreality]: http://rentersreality.com

This module really speaks to the flexibility of the framework. With 
about 8 lines of code I (the end user) was able to add tightly 
integrated commenting to my yesod site. Comments are stored in *my* 
database and users are authenticated using *my* authentication model. 
Furthermore, I (the developer) was able to put together a system that 
integrates this way even with my limited haskell-foo thanks to the 
openness and robustness of existing modules like `yesod-auth` and 
`yesod-persistent`.

I decided to make a few other changes along with this one. First, I 
moved the site from sqlite to postgresql. I originally went with sqlite 
because I was intimidated by postgres. After moving rentersreality to 
postgres, I realized it was no harder to get set up and offered great 
benefits in speed, reliability and maintenance tools. Secondly, I had to 
ditch my simple auth setup (one manually added user) in favor of a more 
typical auth setup. Now, anyone can authenticate with their open id (to 
be able to comment) and I just maintain an `isAdmin` flag manually to 
allow me to do the me-only stuff.

## Why did you do it?

For the sake of completeness here are the Pros and Cons of the recent 
shift:

**Pros:**

* I wrote it.

Since I develop the comments module, I know that I'll get preferential 
treatment when it comes to bug fixes and features. Also, it's awesome.

* Not javascript.

Pages load faster and usage is clean, pure GET and POST html-forms.

* Markdown

Comments are parsed by Pandoc the same way my post content itself is. 
This means that you have the full expressive power of this awesome 
markdown system (any non dangerous html is possible plus syntax 
highlighting and other nice features).

* Integration

Both visually and architecturally, the comments are deeply integrated 
into my site. This is in spite of the code itself being completely 
modularized and reusable anywhere. Yay haskell.

**Cons:**

* I lose all existing comments

I have a plan for this.

* No Quote, or Reply functionality.

I kind of sidestepped the whole javascript edit-in-place usage scenario 
and instead created a full-blown management sub-site for commenting 
users. By following the "comments" link in my sidebar you can see all 
the comments you've left on this site and edit/delete them at-will. It's 
a really clean method which provides a lot of functionality while being 
a) javascript-free and b) still completely modularized out of my 
specific site and reusable anywhere.

Quote could be done with some javascript to just populate the box with a 
markdown blockquote for you. Reply (and the implied threading) would 
require a re-engineering that I might not be willing to go through for a 
while.

* No notifications or rss services

I'm not sure how much of a use there is for this on a site like mine but 
with the new administration sub-site it would be almost trivial to add 
this functionality -- maybe I'll do this soon.

## To existing commenters

If you've commented on this site before **I want to restore your 
comments**, but I need your help.

What I need you to do is go ahead and login once, choose a username, and email
it to me along with the disqus account you commented on previously.

If you commented on the old-old system, I could still restore your 
comments, you'll just have to decide the best way to let me know what 
comments they were (the username you used or the thread/nature of the 
comments, etc).

With this information, I'll be able to reinstate your comments and link 
them to your new identifier on the site.

I hope you'll help me with this; but if not, I understand.

## Play around

So go ahead and use this page to try out the commenting system. See what 
kind of markdown results in what.

If you want any comments (here or on other pages) edited or removed, I 
can always be reached by email. I don't mind running a quick sql 
statement on your behalf.

Let me know of any bugs you find but don't worry about css-fails, those 
should get fixed almost immediately (I just need the content present to 
find them).
