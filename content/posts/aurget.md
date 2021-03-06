---
title: Aurget
date: 2009-12-05
tags: [arch]
---

A simple pacman-like interface to the AUR written in bash.

## About

Aurget is designed to make the AUR convenient and speed up tedious actions. The
user can decide to search, download, build, and/or install packages consistently
through a configuration file or dynamically by passing arguments on the
command-line.

Sourcing user-created PKGBIULDs comes with risks. Please, if you're worried
about this, be sure to view all PKGBUILDs before proceeding.

You have been warned.

## Installation

Study the Arch [wiki][], then manually build and install [aurget][].

[wiki]: https://wiki.archlinux.org/index.php/AUR
[aurget]: https://aur.archlinux.org/packages/aurget/

Follow development via [GitHub][repo].

[repo]: https://github.com/pbrisbin/aurget

## Usage

See `aurget --help`, `man 1 aurget`, and `man 5 aurgetrc`.

## Reporting Bugs

If you've found a bug or want to request a feature, please let me know via
GitHub Issues. If you can implement what you're looking for, please open a Pull
Request, preferably including tests.

Aurget does not and will not search or install from the official repositories.
This is by design and will not be implemented even if you offer a patch. Use
another AUR Helper if this is what you're looking for.
