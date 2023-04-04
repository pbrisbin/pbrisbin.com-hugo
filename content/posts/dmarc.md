---
title: "DMARC"
date: 2023-04-04T13:21:44-04:00
tags: []
---

I recently had an exchange with a security researcher who ethically disclosed
what they considered to be a vulnerability in my [Restyled][] side project. The
vulnerability was that I had not configured DMARC, an email-security mechanism
that I will explain shortly.

Now, Restyled has exactly zero funds. I could not pay bounties if I wanted to,
but I spent some time researching this issue and I've concluded it is not
something that should be considered a vulnerability.

So here is 500 words or so on what I dug up to bring me to this conclusion, as
it may be useful to others facing similar conversations. Please do remember: I
am not ~~a lawyer~~ an authority on any of this and could be entirely mistaken.

**Your risks are your own**.

[restyled]: https://restyled.io

## Securing Email

Email is a plain-text format. Therefore, anyone can send an email with whatever
they want as the `From:` attribution. It's a bit like me wearing a t-shirt that
says "I am Tom Cruise" on it. Am I? You'll never know.

Just like, if you were to believe my shirt, you may give me a multi-million
dollar movie deal that you should not, an email with a "spoofed" From address
might trick you into providing sensitive data to some website you didn't expect.

There are many layers to such an attack, but the availability of a
tom-cruise-affirming t-shirt is one vulnerability in the chain that is worth
reporting, mitigating, and paying bounties for.

As is tradition, we in this industry have to understand and wield multiple,
interrelated acronyms to achieve our goals. In this case, those goals are to
authenticate our emails so users know its us. There are technical references for
all of these that you could find, but I trust the folks at CloudFlare who happen
to have this lovely-- and succinct --primer:

> DKIM and SPF can be compared to a business license or a doctor's medical
> degree displayed on the wall of an office — they help demonstrate legitimacy.
>
> Meanwhile, DMARC tells mail servers what to do when DKIM or SPF fail, whether
> that is marking the failing emails as "spam," delivering the emails anyway, or
> dropping the emails altogether.
>
> -- https://www.cloudflare.com/learning/email-security/dmarc-dkim-spf/

In other words, DKIM & SPF are here to tell you that, no, I am not in fact Tom
Cruise while DMARC is here to say, "if you come upon this imposter, don't even
let him sit down at the movie-deal negotiations."

The analogy gets a bit strained here, but basically DKIM and SPF are used to
prove authenticity and DMARC comes in to answer, _what then?_

## Real User Impact

Without DMARC, unauthenticated emails may reach your INBOX. However, provided
DKIM and are in place, the situation is pretty clear in all the clients I've
tested. For example in Gmail:

![](/images/dmarc/gmail.png)

[Source](https://www.ghacks.net/2016/08/11/gmail-question-marks-unauthenticated-senders/).

Restyled is not a bank, it doesn't handle payments. Its only authentication
methods are 3rd party OAuth, so it lacks its own password-handling flow or code.
We send no automated emails. Our security posture is such that an email claiming
to be from us reaching an INBOX, but clearly looking forged, is not a
high-priority issue to remediate.

## Industry Thoughts

The security researcher in question cited Bugcrowd, and that fact that lack of
DMARC is a P4 vulnerability. That's not very high, but it's high enough that--
based on that alone --I imagine it should register as a concern and be worth a
bounty. This is entirely reasonable.

Ideally, there'd be a page about this specific issue from Bugcrown explaining
their thinking. I cannot find such a page. What I can find is the following:

- [A generic page on "Email Spoofing"][spoofing] which talks about DKIM, SPF,
  and DMARC altogether. This doesn't help prioritizing the case of DKIM/SPF in
  place, but DMARC missing.
- A number of project-specific pages like [this one][stackpath] that explicitly
  say vulnerabilities about any of DKIM, SPF, or DMARC (not just DMARC) are "out
  of scope" meaning they won't pay bounties. Interesting.

[spoofing]: https://www.bugcrowd.com/glossary/email-spoofing/
[stackpath]: https://bugcrowd.com/stackpath/updates/919ea3cc-b5db-4980-aac2-7b70abe11513

https://github.com/bugcrowd/vulnerability-rating-taxonomy/issues/195

https://www.bugcrowd.com/blog/bugcrowd-releases-vulnerability-rating-taxonomy-1-6/
