---
title: Pacprune
date: 2011-06-11
tags: [arch]
---

A fairly long time ago, there was a thread on the Arch forums about 
clearing your pacman cache.

Pacman's normal `-Sc` will remove all versions of any packages that are 
no longer installed and `-Scc` will clear that plus old versions of 
packages that *are* still installed.

The poster wanted a way to run `-Scc` but also keep the last 1 or 2 
versions back from installed. There was no support for this in pacman 
directly, so a bit of a `bash`-off ensued.

I wrote a pretty crappy script which I posted there, it laid around in 
my `~/.bin` collecting dust for a while, but I recently rewrote it. I'm 
pretty proud of the result for its effectiveness and succinctness, so I 
think it deserves a little discussion.

The methodology of the two versions is the same, but this new version 
leans heavily on good ol' unix shell-scripting principles to provide the 
exact same functionality in way less code, memory, and time.

## Approach

The first approach discussed on the thread was to parse filenames for 
package and version, then do a little `sort`-`grep`ping to figure out 
which versions to keep and which versions to discard. This method is 
fast, but provably inaccurate if a package name contains numbers on the 
end.

I went a different way.

For each package, pull the `.PKGINFO` file out of the archive, parse the 
`pkgname` and `pkgversion` variables out of it, then do the same 
`sort`-`grep`ping to figure out what to discard.

My first implementation of this algorithm was really bad. I'd parse and 
write `pkgname|pkgversion` to a file in `/tmp`. Then I'd `grep` unique 
package names using `-m` to return at most the number of versions you 
want to keep (of each package) and store that in another file. I'd then 
walk those files and `rm` the packages.

Ick.

## Needs moar unix

The aforementioned ugliness, plus some configuration and error checking 
weighed in at 162 lines of code, used two files, and was dirt slow. I 
decided to re-attack the problem with a unix mindset.

In a nutshell: write small units that do one thing and communicate via 
simple text streams.

The first unit this script needs is a parser. It should accept a list of 
packages (relative file paths) on `stdin`, parse and output two 
space-separated values on `stdout`: name and path. The path will be 
needed by the next unit down the line, so we need to pass it through.

```bash 
parse() {
  local package opt

  while read -r package; do
    case "$package" in
      *gz) opt='-qxzf' ;;
      *xz) opt='-qxJf' ;;
    esac

    bsdtar -O $opt "$package" .PKGINFO |\
        awk -v package="$package" '/^pkgname/ { printf("%s %s\n", $3, package) }'
  done
}
```

11 lines and damn fast. Thank god for `bsdtar`'s `-q` option. It tells 
the extraction to stop after finding the file I've requested. Since the 
`.PKGINFO` file is usually the first thing in the archive, we barely do 
any work to get the values.

It's also done completely in RAM by piping `tar` directly to `awk`.

Step two would be the actual pruning. Accept that same space-separated 
list on `stdin` and for any package versions *beyond* the ones we want 
to keep (the 3 most recent), echo the full path to the package file on 
`stdout`.

```bash 
prune() {
  local name package last_seen='' num_seen=0

  while read -r name package; do
    [[ -n "$last_seen" ]] && [[ "$last_seen" != "$name" ]] && num_seen=0

    num_seen=$((num_seen+1))

    # print full path
    [[ $num_seen -gt $versions_to_keep ]] && readlink -f "$package"

    last_seen="$name"
  done
}
```

Just watch the list go by and count the number of packages for each 
name. I'm ensuring that the list is coming in reverse sorted already, so 
once we see the number of packages we want to keep, any same-named 
packages after that should be printed.

So simple.

{{< well >}}
This function can get away with being simple because it doesn't take 
into account what's actually installed on your system. It just keeps the 
most recent 3 versions of each unique package in the cache. Therefore, 
to do a full clean, run `pacman -Sc` first to remove all versions of 
uninstalled software. Then use this script to clear all but installed 
plus the two previous versions. This assumes the highest version in the 
cache is the installed version which may or may not be true in all 
cases.
{{< /well >}}

All that's left is to make that reverse sorted list and pipe it through.

```bash 
find ./ -maxdepth 1 -type f -name '*.pkg.tar.[gx]z' | LC_ALL='C' sort -r | parse | prune
```

So the whole script (new version) weighs in at ~30 lines (with 
whitespace) and I claim it is exactly as feature-rich as the first 
version.

I know what you're saying: there's no definition of the cache, no 
optional safe-list vs actual-removing behavior, there's no removing at 
all!

Well, you're just not thinking unix.

```bash 
$ cd /some/cache/of/packages
$ pacprune                  # as a normal user, just print the list that 
                            # should be removed -- totally safe.
$ pacprune | sudo xargs rm  # then do the actual removal
```

You're free to get as fancy as you'd like too...

```bash 
$ archiveit() { sudo mv "$@" ~/pkg_archive/; }
$ pacprune | xargs archiveit
```

And the only configuration is setting the `versions_to_keep` variable at 
the top of the script.

The script can be found in my [scripts repo][repo].

[repo]: https://github.com/pbrisbin/scripts/blob/master/pacprune
