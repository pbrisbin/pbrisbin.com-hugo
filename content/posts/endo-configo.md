---
title: "Endo Configo"
date: 2022-08-01T15:39:53-04:00
tags: ["haskell"]
---

Semigroups are, quite possibly, my favorite part of Haskell. I'm constantly
surprised how much business logic can be expressed by a single `foldMap` using
the correct Semigroup. My co-worker Evan has [said as much][aggregations], and
better.

[aggregations]: https://tech.freckle.com/2017/09/22/aggregations/

One perhaps unfamiliar Semigroup is `Endo`. Don't let the heady name confuse you
too much, it's a `newtype` over `(a -> a)`, any non-type-changing function. It
forms a `Semigroup` where `mappend` is `(.)` and `mempty` is `id`. In other
words, combining two `Endo`s `f <> g` is `f . g`, and since anything combined
with `mempty` must be equivalent to itself, `id` is the only choice there.

Why is this interesting? Well, I'm glad you asked.

## A Structural Approach to Configuration Parsers

Parsing configuration data from a configuration file, command-line options,
environment variables, or all three is a common need for any application. We'll
use configuration of "log settings" via environment variables and
[`envparse`][envparse] as a running example:

[envparse]: https://hackage.haskell.org/package/envparse

```hs
data LogSettings = LogSettings
  { settingsLogLevel :: LogLevel
  , settingsLogColor :: LogColor
  }

envLogSettings :: Parser LogSettings
envLogSettings = LogSettings
  <$> var readLogLevel "LOG_LEVEL" (def LevelInfo)
  <*> var readLogColor "LOG_COLOR" (def LogColorAuto)

readLogLevel :: String -> Either String LogLevel
readLogLevel = undefined

-- and so on...
```

This is what I'd call a _structural_ approach: you encode the shape of your
configuration as a record type, then parse it using a neat Applicative stanza
that matches the same structure. `aeson` and `optparse-applicative` can be used
to make similar parsers as those configuration inputs are added to your
application.[^tangent]

The wart here is when you want to add something like `DEBUG=true` as an
additional way to change `LOG_LEVEL`. Now the configuration source structure and
the desired data structure differs. One way or another this mismatch needs to be
dealt with, and it has always come out slightly ugly in my experience.

Perhaps there's a better way...

## A Functional Approach with Endo Parsers

One pattern I tried (and loved) in a recent module was to write a more
_functional_ parser. It plays really well with some other patterns already in
place in the application. For example, we were hiding the details of the
configuration record, to avoid requiring any major version bumps as we evolve
the internals:

```hs
data LogSettings = LogSettings
  { settingsLogLevel :: LogLevel
  , settingsLogColor :: LogColor
  }

defaultLogSettings :: LogSettings
defaultLogSettings = undefined

setLogLevel :: LogLevel -> LogSettings -> LogSettings
setLogLevel = undefined

-- and so on...
```

Constructing a `LogSettings` by hand would follow a familiar pattern:

```hs
setLogLevel x $ setLogColor y $ defaultLogSettings
```

With this usage in mind, I built a parser that parses functions instead of
values. By parsing them wrapped in `Endo`, we can leverage this great Semigroup
for the combine and apply step:

```hs
envLogSettings :: Parser (Endo LogSettings)
envLogSettings = fold <$> sequenceA
  [ var (endo setLogLevel readLogLevel) "LOG_LEVEL" (def mempty)
  , var (endo setLogColor readLogColor) "LOG_COLOR" (def mempty)
  ]
```

It wouldn't be too bad to in-line `endo`, but I extracted a helper anyway:

```hs
endo
  :: (x -> a -> a)
  -- ^ How to update an @a@ given some @x@
  -> (String -> Either String x)
  -- ^ How to parse that @x@ from a 'String' variable value
  -> Either String (Endo a)
  -- ^ An 'Endo' that holds that update
endo setter reader x = Endo . setter <$> reader x
```

We're parsing an `Endo` now, so we have to un-wrap it and apply it to
`defaultLogSettings` at some point. This is a bit noisy and a non-obvious
choice, and you could certainly "dispatch" the `Endo` within `envLogSettings`
easily, but there will be a payoff for this approach later:

```hs
main :: IO ()
main = do
  logSettings <- ($ defaultLogSettings) . appEndo
    <$> Env.parse id envLogSettings

  runApp App.run logSettings
```

One benefit with this construction is that the defaults are centralized in
`defaultLogSettings`. In the _structural_ parser, we had individual defaults in
each `var` expression; this would either duplicate a `defaultLogSettings` value
or require further extractions of (e.g.) `defaultLogLevel` values, to avoid bugs
via said duplication.

But the real benefit here is when we introduce `DEBUG`...

```diff
 envLogSettings :: Parser (Endo LogSettings)
 envLogSettings = fold <$> sequenceA
   [ var (endo setLogLevel readLogLevel) "LOG_LEVEL" (def mempty)
   , var (endo setLogColor readLogColor) "LOG_COLOR" (def mempty)
+  , var (flag mempty (Endo $ setLogLevel LevelDebug)) "DEBUG" mempty
   ]
```

👌

Now that our source "structure" is a list of functions, there's no overall
change to the design when you need to parse a new "modify the config in some
arbitrary way" value.

## Introducing Other Sources

As mentioned in a footnote, you could extend the _structural_ parser with
wrappers on each field of the record to nicely parse multiple sources and
combine them with desired semantics (e.g. `Last`). The same principle applies
here, but it's all a bit simpler with one wrapped function instead of records or
wrapped values:

```hs
-- optparse-applicative

optLogSettings :: Parser (Endo LogSettings)
optLogSettings = fold <$> sequenceA
  [ option (eitherReader (endo setLogLevel readLogLevel)) (long "log-level")
  , flag mempty (Endo $ setLogColor LogColorAlways) (long "color")
  , flag mempty (Endo $ setLogLevel LevelDebug) (long "debug")
  ]
```

Since `envparse` was inspired by `optparse-applicative`, these parsers look a
lot alike (we can even re-use `endo`). Back in `main`, we can see why the choice
to leave the `Parser`(s) at `Endo` is helpful:

```hs
main :: IO ()
main = do
  logSettings <- (<>)
    <$> Env.parse id envLogSettings
    <*> Options.parse optLogSettings

  let settings = appEndo logSettings $ defaultLogSettings

  runApp App.run logSettings
```

[^tangent]: Semigroups could come in handy here generally, such as using `Last`
  or `First` in these record fields to determine how different sources override
  each other under `(<>)`-composition. That's not the Semigroup we're looking
  for, so we'll avoid the tangent for now.
