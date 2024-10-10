---
title: "Wai Middleware"
date: 2024-10-10T09:22:03-04:00
tags: []
---

No matter what web framework you use in Haskell, you will almost certainly
target WAI, the [Web Application Interface][wai], when running it. Not only is
this an easy and convenient choice, but implementing functionality as a WAI
middleware means that it can be re-used, regardless of framework. This means
neither Yesod, Servant, nor Scotty need to re-invent request logging, timeouts,
https-redirection, or a whole [host of other things][wai-middleware].

[wai]: #todo
[wai-middleware]: #todo

However, the `Application` and `Middleware` types used by WAI can be confusing,
and writing middleware is one place you will encounter them. They are merely
synonyms for continuation-passing functions of requests and so are composed by,
well, function composition. The simplicity is great, but it can make reasoning
about the order behaviors under said composition hard.

In this post, I'm going to show some skeleton code for middlewares
(middles-ware?), calling out three points where it's possible to add behavior.
We'll then see how those behaviors interact under composition. Finally, I'll use
what we discover to write a taxonomy of what middleware can do and prescribe a
way to order middleware relative to others based one where they fall.

{{< details "Expand to see pre-amble needed to run the examples" >}}

```hs
#!/usr/bin/env stack
-- stack script --snapshot lts-22.37 --package http-types --package wai --package warp

{-# LANGUAGE OverloadedStrings #-}

module Main where

import Network.Wai
import Network.Wai.Handler.Warp qualified as Warp
import Network.HTTP.Types.Status (status200)
import System.Environment (getEnv)
```

{{< /details >}}

## A Simple Application

In order to test our middleware, we'll need an `Application`. A WAI
`Application` is just a function that takes a `Request` and a "respond"
function, which you are meant to call when you want to respond:

```hs
type Application
  = Request                            -- Given this,
  -> (Response -> IO ResponseReceived) -- you must call this,
  -> IO ResponseReceived               -- as the only way to return this
```

We'll make our application as simple as possible and just respond "OK" to all
requests:


```hs
app :: Application
app _request respond = do
  putStrLn "app: about to respond"
  recvd <- respond $ responseLBS status200 [] "OK\n"
  putStrLn "app: responded"
  pure recvd
```

If we didn't need to add that Observablity™️, we could've simply wrote:

```hs
app :: Application
app _request respond = respond $ responseLBS status200 [] "OK\n"
```

Or get even cuter:

```hs
app :: Application
app = const ($ responseLBS status200 [] "OK\n")
```

Anyway, moving on.

## De-constructed Middleware

The `Middleware` type is also quite elegant; it's just a function that turns one
`Application` into another.

```hs
type Middleware = Application -> Application
```

Expanding the `Application` synonyms breaks my brain and I don't recommend it.
Instead, I find the thing is easier to _use_ than to understand. Let's make one:

```hs
noop :: Middleware
noop app request respond = undefined
--   ^   ^       ^
--   |   |       |
--   |   |       ` A function you'll call to get a ResponseReceived
--   |   |
--   |   `-- A Request
--   |
--   `-- Some other Application
```

What's the simplest thing we can do? Call the `Application` with the `Request`
and respond function given:

```hs
noop :: Middleware
noop app request respond = app request respond
```

Which, perhaps unsurprisingly, is just `id`:


```hs
noop :: Middleware
noop = id
```

This is fun and all, but how do we do things? Well, first let's "de-construct"
our `noop` middleware with a bunch of unnecessary functions and bindings:

```hs
noop :: Middleware
noop app request respond = do
  recvd <- app request $ \response -> do
    recvd <- respond response
    pure recvd
  pure recvd
```

Before proceeding, convince yourself that this is equivalent.

What we've done is exposed seams between the various applications that are
happening. Let's add more oBsErVaBiLiTy at each of these seams:

```hs
middleware :: String -> Middleware
middleware name app request respond = do
  putStrLn $ "middleware " <> name <> ": seeing request" -- (1)

  recvd <- app request $ \response -> do
    putStrLn $ "middleware " <> name <> ": about to respond" -- (2)

    recvd <- respond response
    putStrLn $ "middleware " <> name <> ": responded" -- (3)

    pure recvd

  putStrLn $ "middleware " <> name <> ": done" -- (4)
  pure recvd
```

These are the four points at which a `Middleware` can _do things_. My goal here
is to lay out what sorts of things make sense to do at which points, set some
limits on doing too many things at once, and finally describe how the sorts of
things a middleware does should inform where you place it under composition with
other middleware doing other sorts of things.

But first, we gotta run the thing.

## Defining Order

Add the following:

```hs
main :: IO ()
main = do
  port <- read <$> getEnv "PORT"
  Warp.run port
    . middleware "1"
    . middleware "2"
    . middleware "3"
    $ app
```

In one terminal run,

```console
% PORT=3000 ./wai-middleware-app
```

In another run,

```console
% curl localhost:3000
OK
```

And back in the first terminal, see what we got.

```console
middleware 1: seeing request
middleware 2: seeing request
middleware 3: seeing request
app: about to respond
middleware 3: about to respond
middleware 2: about to respond
middleware 1: about to respond
middleware 1: responded
middleware 2: responded
middleware 3: responded
app: responded
middleware 3: done
middleware 2: done
middleware 1: done
```

As you can see, the idea of order is ambiguous:

- There are two cases the middleware runs 1-2-3 and two cases it runs 3-2-1
- The `app` is the "last" to see the request, but "first" to respond

This is why composing middles-ware correctly is confusing. It depends at which
point in the life-cycle the middleware's behavior matters to the things with
which it's composed.
