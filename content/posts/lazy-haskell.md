---
title: Lazy Haskell
date: 2011-04-09
tags: [haskell, xmonad]
---

Let's say that you have a list of values and you needed to check if any 
of those values satisfied some condition.

This can be solved easily with Haskell's `any` function, but let's say 
you didn't have that.

Here's an alternative method using `foldr`.

```haskell 
any' :: (a -> Bool) -> [a] -> Bool
any' p list = foldr ((||) . p) False list

any' (== 'x') ['x','y','z']
-- True

any' even [1,3,5,7]
-- False
```

To understand how this works, you've got to wrap your head around two 
non-trivial functions for haskell beginners: `foldr` and `.`.

## Dot

`.` takes two functions and *composes* them to create a new function. 
This is best seen by way of example.

If you know that `length` counts items in a list and `even` tests if a 
number is even, then when someone asks you to write a function that 
determines if the number of characters in a string is even, it should be 
little more than these two words put together some way...

```haskell 
-- types refresher:
-- 
-- String = [Char]
-- length :: [a] -> Int
-- even   ::        Int -> Bool
-- 

-- correct, but ugly
stringIsEven :: String -> Bool
stringIsEven s = even (length s)

-- better, but too long
stringIsEven s = even $ length s

-- perfect
stringIsEven = even . length
```

{{< well >}}
Remember, `$` is function *application* while `.` is function 
*composition*.
{{< /well >}}

I think that gives you a general idea for how it works. Now let's 
translate that to the specific example at hand:

`(||)` is just another function. Think about that for a minute. Welcome 
to haskell.

`(||)` takes two `Bool`s and returns a `Bool`; `True` if either one of 
its arguments is `True`.

```haskell 
(||) True False
-- True

(||) False False
-- False
```

{{< well >}}
Curious how `(||)` is defined in haskell's Prelude?

```haskell 
(||) :: Bool -> Bool -> Bool
True  || _ = True
False || x = x
```

Wow. 

Haskell's laziness means there's no special tricks needed to make if 
statements "short circuit". Haskell won't evaluate the second expression 
if the first is `True` because it's simply never needed.
{{< /well >}}

OK, back to our function.

`p` is provided as the first argument to our `any'` function and we know 
that it's type is `(a -> Bool)`. This means it has to be a test that 
will check a value and return `True` or `False`.

So, what might the type of `((||) . p)` be?

This *composed* function (and you've really got to think of it as *one* 
function) will take some value, `a` as its first argument. It will apply 
`p` to it which gives an intermediate `Bool`. That `Bool` is then passed 
through as the first argument to `(||)`.

`(||)`, having gotten its first argument already, only needs one more 
argument. Since it's not supplied by anything else, it's now required as 
an argument to the composed function.

```haskell 
-- suppose p is already defined like so:
p :: Char -> Bool
p = (== 'z')

-- types refresher:
-- 
-- p    :: Char -> Bool
-- (||) ::         Bool -> Bool -> Bool
-- 

((||) . p) 'z' False
-- True

((||) . p) 'x' True
-- True


-- ((||) . p) :: Char -> Bool -> Bool
```

Easy, right?

## Folds

The next crazy function is `foldr`. A *fold* in the general sense is a 
way to reduce a list.

If you've got *a list of items*, a *reducing function*, and some 
*initial value*, then a *fold* is the process of using these three 
things to reduce the list to a single value.

This can be seen in the type of `foldr`. A great deal of information can 
be learned in haskell by simply taking a look at types; that's why 
[haddocks][folds] are so invaluable.

```haskell 
foldr :: (a -> b -> b) -- ^ a reducing function
      -> b             -- ^ some initial value
      -> [a]           -- ^ a list of items
      -> b             -- ^ the resultant single value
```

Take care to note the type of the reducing function.

It must accept as its first argument the same type as your list of items 
contains and as its second argument the same type as your initial value.

Its result is also the same type as your initial value. This is 
important because the initial value *and* the result of the previous 
application of `foldr` must be the same type if we want the required
recursion to be type safe.

Often, the types `a` and `b` are the same (as in `sum'` explained 
below), but this is not required.

{{< well >}}
`foldr` and `foldl` are different in the direction of the fold: folding 
to the right or folding to the left. In some cases this doesn't matter, 
in others it does.
{{< /well >}}

Let's look at a folding sum as a concrete example:

```haskell 
sum' :: [Int] -> Int
sum' xs = foldr (+) 0 xs

sum' [1,2,3,4,5]
-- 15

-- how's it work?
foldr (+) 0 [1,2,3,4,5]
-- 15

-- the reducing function is applied with its second argument as the 
-- initial value
result     = (+) 1 0  -- 1 + 0  = 1

-- now we apply the same function but use the result of the previous 
-- application as the new initial value and act on the next element
result'    = (+) 2 1  -- 2 + 1  = 3

-- rinse and repeat until all elements are used up
result''   = (+) 3 3  -- 3 + 3  = 6
result'''  = (+) 4 6  -- 4 + 6  = 10
result'''' = (+) 5 10 -- 5 + 10 = 15

((((0 + 1) + 2) + 3) + 4) + 5 
-- 15
```

Here's another breakdown with the recursion explicitly shown rather than 
the values it represents:

```haskell 
foldr (+) 0 [1,2,3,4,5]

result     =                                 (+) 1 0
result'    =                         (+) 2 $ (+) 1 0
result''   =                 (+) 3 $ (+) 2 $ (+) 1 0
result'''  =         (+) 4 $ (+) 3 $ (+) 2 $ (+) 1 0
result'''' = (+) 5 $ (+) 4 $ (+) 3 $ (+) 2 $ (+) 1 0
-- 15
```

This is an easy example where you can see clearly how things work out. 
In our case it's a bit more complex, but the principle is the same:

```haskell 
-- assume p is defined like so
p :: Char -> Bool
p = (== 'b') 


foldr ((||) . p) False ['a','b','c']
-- True

-- value breakdown:
result   = ((||) . p) 'a' False -- (== 'b') 'a' || False = False
result'  = ((||) . p) 'b' False -- (== 'b') 'b' || False = True   DING!
result'' = ((||) . p) 'c' True  -- (== 'b') 'c' || True  = True

-- recursion breakdown:
result    =                                   ((||) . p) 'a' False
result''  =                  ((||) . p) 'b' $ ((||) . p) 'a' False -- <- this is the only
result''' = ((||) . p) 'c' $ ((||) . p) 'b' $ ((||) . p) 'a' False --    line ever evaulated
-- True
```

So the whole thing reduces to True, just as we'd expect.

## Why?

This was a really slow and deliberate explanation. I did it this way 
because I had a real-world situation where I had to come to this exact 
understanding to solve a problem. OK, not really to *solve* some dire 
*problem* per say, but to do something I wanted to do in an elegant 
way...

I wanted to walk you all through it from the start only so someone 
not-so-familiar with haskell might a) see its beauty and b) actually 
understand the single line of code I'm going to show you in a few more 
paragraphs.

Sorry.

In my window manager of choice, XMonad, there is a means to test a 
window's property and take some action depending on the result.

The simplest example is *move windows with class "firefox" to the "web" 
workspace*.

```haskell 
className =? "firefox" --> doShift "web"
```

Easy.

There's also a means to *OR* rules like these together.

With this, I can say *move windows with class "firefox" OR title 
"chrome" to the "web" workspace*.

```haskell 
className =? "firefox" <||> title =? "chrome" --> doShift "web"
```

The two functions `(=?)` and `(<||>)` behave *exactly* like their normal 
`(==)` and `(||)` counterparts. They're just *lifted* into a *Query 
Monad*. This is a concept you don't need to comprehend right now, just 
know that there's no elegant way to apply `any` in this context.

That made it difficult to write a simple function: `matchAny` that could 
be a test if *any* of a window's properties (class, title, name, or 
role) matched the test string.

Now the `any'` exercise isn't looking so unrealistic, is it?

```haskell 
-- types refresher:
-- 
-- any :: (a -> Bool) -> [a] -> Bool
-- 
-- we need the same thing, just lifted to the "Query" context:
-- liftAny :: (a -> Query Bool) -> [a] -> Query Bool
-- 

-- our any reimplimentation from ealier:
any'    p list = foldr ((||)   . p)         False  list

-- almost identical:
liftAny p list = foldr ((<||>) . p) (return False) list
```

{{< well >}}
`return False` is simply the *lifted* version of `False` just like 
`(=?)` is the *lifted* version of `(==)`...
{{< /well >}}

Now my manage hooks can leverage a [list comprehension][listcomp] for a 
much more concise and readable rule.

```haskell 
matchAny :: String -> Query Bool
matchAny s = liftAny (=? s) [className, title, name, role]

myManageHook = composeAll [ matchAny s --> action | (s, action) <- myActions ]

    where

        myActions = [ ("rdesktop"  , doFloat         )
                    , ("Xmessage"  , doCenterFloat   )
                    , ("Gmrun"     , doCenterFloat   )
                    , ("Uzbl"      , doShift "2-web" )
                    , ("Uzbl-core" , doShift "2-web" )
                    , ("Chromium"  , doShift "2-web" )
                    , ("irssi"     , doShift "3-chat")
                    ]
```


## Finally

So why is this post about laziness?

```haskell 
foldr ((||) . (== True)) False [False, False, True, undefined, undefined]
-- True
```

That statement "short circuits". That's only possible because of lazy 
evaluation.

[folds]: http://www.haskell.org/ghc/docs/7.0.2/html/libraries/base-4.3.1.0/Data-List.html#g:3 "folds explained"
[listcomp]: http://www.haskell.org/haskellwiki/List_comprehension "List comprehensions explained"
