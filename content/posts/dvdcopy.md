---
title: Dvdcopy
date: 2009-12-05
tags: [shell]
---

{{< well >}}
Do not use this for bad things, m'kay?
{{< /well >}}

## What it looks like

![Dvdcopy Shot](/images/dvdcopy/dvdcopy.png)

## Usage


    usage: dvdcopy [ --option(=<argument>) ] [...]

    ~/.dvdcopy.conf will be read first if it's found (even if --config
    is passed). for syntax, see the help entry for the --config option.
    commandline arguments will overrule what's defined in the config.

    invalid options are ignored.

    options:

      --config=<file>               read any of the below options from a
                                    file, note that you must strip the
                                    '--' and set any argument-less
                                    options specifically to either true
                                    or false

                                    there is no error if <file> doesn't
                                    exist

      --directory=<directory>       set the working directory, default
                                    is ./dvdcopy

      --keep_files                  keep all intermediate files; note
                                    that they will be removed the next
                                    time dvdcopy is run regardless of
                                    this option

      --device=<file>               set the reader/burner, default is
                                    /dev/sr0

      --title=<number>              set the title, default is longest

      --size=<number>               set the desired output size in KB, 
                                    default is 4193404

      --limit=<number>              set the number of times to attempt a
                                    read/burn before giving up, default
                                    is 15

      --mpeg_only                   stop after transcoding the mpeg
      --dvd_only                    stop after authoring the dvd
      --iso_only                    stop after generating the iso

      --mpeg_dir=<directory>        set a save location for the
                                    intermediate mpeg file, default is
                                    blank -- don't save it

      --dvd_dir=<directory>         set a save location for the
                                    intermediate vob folder, default is
                                    blank -- don't save it

      --iso_dir=<directory>         set a save location for the
                                    intermediate iso file, default is
                                    blank -- don't save it

      --mencoder_options=<options>  pass additional arbitrary arguments
                                    to mencoder, multiple options should
                                    be quoted and there is no validation
                                    on these; you'll need to know what
                                    you're doing. the options are placed
                                    after '-dvd-device <device>' but
                                    before all others

      --quiet                       be quiet
      --verbose                     be verbose

      --force                       disable any options validation,
                                    useful if ripping from an image file

      --help                        print this

## What's it do?

Pop in a standard DVD9 (\~9GB) and type `dvdcopy`. The script will
calculate the video bitrate required to create an ISO under 4.3GB
(standard DVD5). It will then use `mencoder` to create an
authorable image and burn it back to a disc playable on any
standard player.

Defaults are sane (IMO), but can be adjusted through the config
file or the options passed at runtime (or both). I've now added a
lot of cool features as described in the help.

## How to get it

Install the AUR package
[here](http://aur.archlinux.org/packages.php?ID=42433).

Grab the source from my git repo
[here](http://github.com/pbrisbin/dvdcopy).
