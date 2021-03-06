---
title: Backups
date: 2010-01-03
tags: [linux]
---

{{< well >}}
This post is very out of date. The scripts which are its subject no 
longer exist as I now use two much simpler scripts which can be found in 
my [scripts repo][scripts].

[scripts]: https://github.com/pbrisbin/scripts "Scripts"
{{< /well >}}

Backups are extremely important. In linux, with a little effort and 
hardrive space, one can easily come up with a fully automated backup 
solution to suit any needs. Here, I'd like to outline my setup. Feel 
free to take it and adapt to your needs.

I'll go through what's required, how and why I do it the way I do, as 
well as the shortcomings of how I'm doing it.

## Requirements

My main box runs on one 500G hardrive. So far, this has suited me well 
even with my extensive movies and music collection. I decided I wanted 
to have a daily backup and a monthly backup and only one copy of each, 
so I went out and got a 1TB hardrive, split it, and now use that for 
both.

All you need is space, so whether you use an internal drive like me, an 
external USB, or some off-site scp/rsync situation is up to you; you'll 
just have to modify my below script(s) to suit your setup.

## How I do it

The first is a backup script that runs via cron daily and monthly. 
~~It can be downloaded from my git repo~~.

The script defines an array of files to include and another to exclude:

```bash 
includes=( /srv/http /home/patrick /etc /usr /var /boot )
excludes=( Downloads lost+found )
```

It takes those directories and just `rsync`s them with the backup 
location:

```
/mnt/backup/daily/
|-- boot
|-- etc
|-- http
|-- patrick
|-- usr
`-- var

/mnt/backup/monthly/
|-- boot
|-- etc
|-- http
|-- patrick
|-- usr
`-- var
```

It also creates two text files: one that lists all your installed 
packages less those that are foreign (from the AUR) and another that 
lists those foreign packages.

These lists can be used to quickly reinstall everything you had 
installed at the time of the backup.

```bash 
pacman -Qqe | grep -Fvx "$(pacman -Qqm)" > "$backup_dir/paclog"
pacman -Qqm > "$backup_dir/aurlog"
```

Another script I use constantly is `retrieve` which will take the 
filenames passed on the commandline and look for them in your backups. 
If found, the files are retrieved and re-inserted into you live system.

This is great if you've seriously screwed up your xorg.conf (something 
not in git) and you want to just roll back to what you had yesterday.

The only trick to it is that it has to handle the fact that my backup 
stores `patrick/` at top level even though it's `/home/patrick/` on the 
live system.

`retrieve` is <del>also</del> no longer available in my git repo.

The last script that I have, I haven't had to use *--knocks on wood--*. 
This `restore` script is intended to be used after a crash **and clean 
re install** to restore your system back from the directories made by my 
backup script.

You guessed it, `restore` is <del>also</del> no longer in the repo.

[restore]: http://github.com/pbrisbin/scripts/blob/master/restore

## Why mine sucks

This solution works for me, but it has its shortcomings. Here are a 
few things to be aware of if you decided to implement something like 
what I have.

## Not off-site, or even out-of-box.

If my apartment burns down, my backups are useless. To mitigate this, 
I've started taking manual copies of my monthly backup and storing them 
on a separate drive in a fireproof box.

## Backups are not rolling

This isn't so bad for the dailies, but my monthly backup occurs every 
month on the first; this means if you have an issue that's more then two 
days old, and you happen to notice on the 2nd, you don't have a backup 
old enough to fix it.

## Untested

I've never had to use `restore`, though I do use `retrieve` all the 
time. Anyone will tell you, an untested backup solution is no solution 
at all. Guess I'm just too lazy to hose my install to test it. Worse 
comes to worst, I know the backed up data is good; if my `restore` 
script fails I can always manually copy everything over. I pretty much 
did this last time I installed a new Arch box; as I tend to reuse 
configs, just grabbing them off of my main box's backups really sped up 
the process.
