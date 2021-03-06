---
title: Dvd2iso
date: 2012-11-15
tags: [ruby]
---

My latest bash-to-ruby rewrite was dvdcopy to dvd2iso. I changed the 
name both to disambiguate, and because my primary use case was no longer 
to duplicate disc to disc, but to just generate the ISO. It's very 
simple to just [burn][] that ISO back to disc if I feel like it.

[burn]: https://github.com/pbrisbin/scripts/blob/master/burn

The benefits of the new script are:

1. Less and simpler code
2. Coded at a higher level
3. Easier to use

There are far less options than dvdcopy; you can choose the device and 
output file, that's it. Though, the script is easy enough to configure 
further by simply editing the source.

```
usage: dvd2iso [options]
    -i, --input DEVICE
    -o, --output FILE
```

The output file can have a `%s` in it which will be replaced by the 
downcased, underscored version of the DVD name.

When the script runs, all subcommands have their output redirected to 
a log file which you're told to consult if there's some error. Instead, 
what you get as output is actually every command the script runs in 
copy-pastable format.

This makes it very easy to rerun any or all of the script's actions 
if you want to tweak or debug something.

**Actual script output**:

```
$ dvd2iso -o 'rips/%s.iso'
#
# Ripping SOME_DVD
#   Title 1, 29 Chapters
#
mkdir -p ./dvd2iso_tmp
mencoder \
  dvd://1 \
  -dvd-device '/dev/sr0' \
  -mc 0 \
  -of mpeg \
  -mpegopts format=dvd:tsaf \
  -oac copy \
  -ovc lavc \
  -vf scale=720:480,pullup,softskip,harddup \
  -lavcopts vcodec=mpeg2video:vrc_buf_size=1835:vrc_maxrate=9800:vbitrate=5824:keyint=18:vstrict=0:aspect=16/9:ilme:ildct \
  -ofps 24000/1001 \
  -o './dvd2iso_tmp/movie.mpeg'
dvdauthor \
  -t \
  -c 00:00:00.000,00:05:54.533,00:10:20.433,00:16:17.533,00:19:56.799,00:24:11.266,00:31:35.866,00:36:28.600,00:37:53.700,00:41:07.067,00:43:30.367,00:47:22.067,00:50:41.700,00:52:27.966,00:55:32.433,00:57:28.100,01:01:05.300,01:03:35.234,01:05:46.634,01:09:14.700,01:11:13.133,01:11:59.299,01:16:17.266,01:19:36.100,01:21:59.533,01:23:34.467,01:26:57.100,01:28:13.767,01:33:49.667 \
  -o './dvd2iso_tmp/MOVIE' \
  './dvd2iso_tmp/movie.mpeg'
dvdauthor \
  -T \
  -o './dvd2iso_tmp/MOVIE'
mkisofs \
  -dvd-video \
  -o './dvd2iso_tmp/movie.iso' \
  './dvd2iso_tmp/MOVIE'
mv ./dvd2iso_tmp/movie.iso rips/some_dvd.iso
rm -r ./dvd2iso_tmp
#
# Success!
#
$
```

Anyway, you can find it in my [bin][]. Enjoy.

[bin]: https://github.com/pbrisbin/scripts/blob/master/dvd2iso
