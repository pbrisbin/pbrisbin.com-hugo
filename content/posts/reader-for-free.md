---
title: "Reader for Free"
date: 2021-09-10T10:39:58-04:00
draft: false
---

This may be obvious or well-known to some, but I discovered the other day that
you can make a `MonadReader env` instance for any `MonadState env m` trivially.
This makes total sense conceptually, since State is just Reader with the extra
ability to modify.

I'm going to talk it through at length, how it works and why I needed it, but if
you're just interested in copying the code (which I know future me will be),
here it is:

```hs
import Control.Monad.Reader
import qualified Control.Monad.State as State

instance MonadReader s (MyThingWithMonadState s) where
    ask = State.get
    local f m = do
        s <- State.get
        State.modify f
        result <- m
        result <$ State.put s
```

Alright, let's slow-walk it now.

## `MonadReader` and `Has`-classes

The `MonadReader` type class allows accessing an "environment" value, called `r`
in documentation, but more often `env` "in the wild". In industry Haskell
applications, there's a pattern emerging to share a single "application
environment" loaded at start-up and then access bits of it abstractly via
`Has`-classes, often called "capabilities".

For example, interacting with a Database might be typed like this:

```hs
fetchThings
    :: ( MonadIO m
       , MonadReader env m
       , HasSqlPool env
       )
    => ThingId
    -> m [Thing]
fetchThings = undefined
```

As your application grows more capabilities, you'll find more `Has` classes on
`env`: `HasRedis` to support queue operations, `HasLogFunc` for logging, etc.

```hs
fetchAndEnqueueThings
  :: ( MonadIO m
     , MonadReader env m
     , HasSqlPool env
     , HasRedis env
     , HasLogFunc env
     )
  => ThingId
  -> m ()
fetchAndEnqueueThings thingId = do
    things <- fetchThings thingId
    logDebug $ "Fetched " <> displayShow things
    traverse_ enqueueThing things

enqueueThing
   :: ( MonadIO m
      , MonadReader env m
      , HasRedis env
      )
   => Thing
   -> m ()
enqueueThing = undefined
```

Aside from clarity (every function's type here shows exactly and only the
effects it needs), one reason this can be useful is that the `m` and its `env`
can vary between implementation and test code. Under test, you might configure a
different environment to suppress logging or intercept API calls to third
parties.

{{< well >}}
👉 There is an alternative to (e.g.) `MonadReader env m, HasLogFunc env` which
is simply `MonadLogger m`. Both styles have their place, and you can even mix
and match; I'm just focusing on the `Has`-style here because this is a
`MonadReader` post.
{{< /well >}}

## Yesod

First let's assume `HasSqlPool` is defined in the idiomatic way:

```hs
class HasSqlPool env where
    sqlPoolL :: Lens' env SqlPool

instance HasSqlPool SqlPool where
    sqlPoolL = id
```

In a Yesod application, you'll give your `App` an instance:

```hs
data App = App
    { appSqlPool :: SqlPool
    , ...
    }

instance HasSqlPool App where
    sqlPoolL = lens appSqlPool $ \x y -> x { appSqlPool = y }
```

Then you can run something like `fetchThings` in `Handler` because it's a reader
of your `App`, right? Well, not really. `Handler` is actually a reader of
`HandlerData child App`, which wraps your `App` up in some Yesod-specific data,
so writing code like,

```hs
getThingsR :: ThingId -> Handler [Thing]
getThingsR thingId = fetchThings
```

Will get you:

```
• Could not deduce (HasSqlPool (HandlerData App App))
```

This is super common, to want some capability from an environment, but find
yourself in some _larger_ environment than the one that has it. Luckily, the
lens-y `Has`-class design is built for solving this scenario.

First, create a lens (or two) to get from the thing you have to the thing you
need. `HandlerData`'s `handlerEnv` is a `RunHandlerEnv child site` and then
`RunHandlerEnv`'s `rheSite` finally gets you the `site`:

```hs
envL :: Lens' (HandlerData child site) (RunHandlerEnv child site)
envL = lens handlerEnv $ \x y -> x { handlerEnv = y }

siteL :: Lens' (RunHandlerEnv child site) site
siteL = lens rheSite $ \x y -> x { rheSite = y }
```

Now we can easily make a "pass-through" instance:

```hs
instance HasSqlPool env => HasSqlPool (HandlerData child env) where
    sqlPoolL = envL . siteL . sqlPoolL
```

This is all well-and-good, but what happens when you want to test `fetchThings`?

```hs
spec :: Spec
spec = withApp $ do
    describe "fetchThings" $ do
        it "runs without erroring" $ do
            fetchThings 42 `shouldReturn` []
```

This'll generate some familiar-looking errors:

```
• Could not deduce (MonadReader env0 (SIO (YesodExampleData site)))
• Could not deduce (HasSqlPool env1)
```

In a Yesod application using `yesod-test`, these specs run in the `YesodExample
site` monad, which is a synonym for `SIO (YesodExampleData site)`. That is an OG
"State over IO" concoction from way back before it was cool:

![sio](/images/reader-for-free/sio.png)

It does not have `MonadReader`, womp. We could open a patch upstream to add an
`MonadReader s (SIO s)` instance easily (it can probably be derived), but what
can we do in the meantime?

Well, unsurprisingly, we do have `MonadState s (SIO s)`, so we could use the
code from the top of this post to make an orphan:

```hs
instance MonadReader s (SIO s) where
    ask = State.get
    local f m = do
        s <- State.get
        State.modify f
        result <- m
        result <$ State.put s
```

Let's talk about how this works: `ask` and `get` are just the same thing, this
is the shared power between Reader and State. Reader's `local` defines how to
run some computation in an altered environment:

```hs
local :: MonadReader r m => (r -> r) -> m a -> m a
```

State's `modify` alters the state/environment, but to mimic the under-powered
Reader's behavior we just have to save it off first and `put` it back after.

We get a different error now though:

```
• Could not deduce (HasSqlPool (YesodExampleData site))
```

We'll this looks familiar! We've implemented `MonadReader` for the `s` in `SIO`,
but we only `HasSqlPool` on `site`. `SIO`'s `s` is actually the `site` wrapped
up in some other, test-specific data as a `YesodExampleData site`, so this
doesn't work. This is exactly the same problem as `HandlerData`.

So let's set up our lens (I'll even use the same name since we're unlikely to
have both in scope at once):


```hs
siteL :: Lens' (YesodExampleData site) site
siteL = lens yedSite $ \x y -> x { yedSite = y }
```

And make another "pass-through" instance:

```hs
instance HasSqlPool site => HasSqlPool (YesodExampleData site) where
    sqlPoolL = siteL . sqlPoolL
```

And there you go 🎉
