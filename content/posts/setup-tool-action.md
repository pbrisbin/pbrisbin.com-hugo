---
title: "Setup Actions Don't Scale"
date: 2022-12-07T17:28:54-05:00
tags: ["github actions"]
---

I see a lot of `this-setup` or `setup-that` GitHub actions around. They all wrap
the [`@actions/tool-cache`][tc] library to install some pre-compiled binary. If
`tool-cache` exists, why do we need all these separate actions? What happens if
I can't find a `-setup` action for the tool I want to use? What if there are
multiple? Do I need to write yet-another `curl | tar` or `gh release download`?

[tc]: #todo

Well, the reason for all these separate `-setup` actions is that
`actions/tool-cache` is fairly low-level. You need to know and provide a bunch
of tool-specific stuff to use it:

- The URL to the archive
- The format of the archive (i.e. gzip or zip)
- If the executables are in some sub-directory

These things vary wildly between tools, and that URL packs a lot of information
in it, like the pet names this project uses to encode OS and Architecture into
the path. So you can't even assume something conventional if the tool uploads
binaries to a GitHub release. The result: `hlint-setup` hard-codes its things
and `setup-pandoc` hard-codes its things. And we all just keep doing this until
we die.

While I was considering writing a `dms-setup-action` to install the [Dead Man's
Snitch Field Agent][dms], I wondered if we could do better. And so I created
[pbrisbin/setup-tool-action][setup-tool] instead. This action attempts to
support every possible `-setup` action's use-case. In other words, if your
intent is to wrap `@actions/tool-cache`, you hopefully won't need to now.

[dms]: https://deadmanssnitch.com/docs/field-agent
[setup-tool]: https://github.com/pbrisbin/setup-tool-action

As a simple first example, here's how I install pre-compiled `weeder` binaries
as part of [freckle/weeder-action][weeder-action]:

[weeder-action]: https://github.com/freckle/weeder-action

```yaml
uses: pbrisbin/setup-tool-action@v1
with:
  name: weeder
  version: ${{ inputs.weeder-version }}
  url: "https://github.com/freckle/weeder-action/releases/download/Binaries/{name}-{version}-{os}-{arch}.{ext}"
```

No separate `weeder-setup` action necessary.

As you might guess, the template-string for `url` allows you to define from the
outside how this particular tool names its pre-compiled binaries. Since I also
maintain weeder-action, I chose to release binaries using `os` and `arch` values
that match Node's `process.platform` and `process.arch` values on GitHub
runners, which is what this template string is filled in with by default.

Of course, there's no consensus in this world. As a slightly more complex case,
let's fully replace [`haskell/actions/hlint-setup`][hlint-setup] for a
hypothetical project that only runs it on `ubuntu-latest`:

[hlint-setup]: https://github.com/haskell/actions/tree/main/hlint-setup

```yaml
with:
  name: hlint
  version: "3.5"
  url: "https://github.com/ndmitchell/{name}/releases/download/v{version}/{name}-{version}-{arch}-{os}.{ext}"
  subdir: "{name}-{version}"
  arch: x86_64
```

This is not too bad, and we've cleanly dealt with three axis of variability:

- HLint has the rarer arch-then-os convention in the URL
- HLint uses `x86_64` instead of the `x64` that `process.arch` gives
- HLint packs its binaries in a sub-directory

The default `os` works on this runner since HLint releases use `linux`, which
matches `process.platform`. For non-Linux runners that's not the case, so if
your project is running HLint on such a platform, you need to override `os` too.
But the values are themselves `os`-specific. To support such cases, all `inputs`
except `name` and `version` can be given at [varying levels of
specificity][specified-inputs]:

[specified-inputs]: https://github.com/pbrisbin/setup-tool-action/blob/c49315d7ccf6271da274e560d8ee9600852c7796/src/inputs.ts#L72-L83

- `<input>-<platform>-<arch>` overrides,
- `<input>-<arch>` overrides,
- `<input>-<platform>` overrides,
- `<input>`

With that in mind, we can make a fully runner-agnostic version of this step:


```yaml
with:
  name: hlint
  version: "3.5"
  url: "https://github.com/ndmitchell/{name}/releases/download/v{version}/{name}-{version}-{arch}-{os}.{ext}"
  subdir: "{name}-{version}"
  os-darwin: osx
  os-win32: windows
  arch: x86_64
```

But surely the `url` won't change by platform, right? Enter Pandoc:

```yaml
with:
  name: pandoc
  version: 2.19.2
  url: "https://github.com/jgm/{name}/releases/download/{version}/{name}-{version}-{os}-{arch}.{ext}"
  url-darwin: "https://github.com/jgm/{name}/releases/download/{version}/{name}-{version}-macOS.zip"
  subdir: "{name}-{version}/bin"
  subdir-win32: "{name}-{version}"
  os-win32: windows
  arch: x86_64
  arch-linux: amd64
```

Yeah, this is a lot. But it's still better than building and maintaining
whole-ass action right? It's also worth noting that if you only ever run things
on one runner, there's no need to get so verbose. Just do what your runner
needs:

```yaml
# ubuntu
with:
  name: pandoc
  version: 2.19.2
  url: "https://github.com/jgm/{name}/releases/download/{version}/{name}-{version}-{os}-amd64.{ext}"
  subdir: "{name}-{version}/bin"

# macOS
with:
  name: pandoc
  version: 2.19.2
  url: "https://github.com/jgm/{name}/releases/download/{version}/{name}-{version}-macOS.zip"
  subdir: "{name}-{version}/bin"

# windows
with:
  name: pandoc
  version: 2.19.2
  url: "https://github.com/jgm/{name}/releases/download/{version}/{name}-{version}-windows-x86_64.{ext}"
  subdir: "{name}-{version}"
```

And what of the goal that spurned this yak? Can it install my Field Agent?

```yaml
with:
  name: dms
  version: latest
  url: "https://releases.deadmanssnitch.com/field-agent/{version}/{name}_{os}_{arch}.{ext}"
  os-darwin: macos
  os-win32: windows
  arch: amd64
```

👌
