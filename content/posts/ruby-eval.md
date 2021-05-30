---
title: Ruby Eval
date: 2011-10-25
tags: [ruby]
---

Ruby's `intance_eval` and `class_eval` are awesome tricks of the 
language that can really cut down on redundant code or let you do 
truly dynamic things that you'd have never thought possible.

There's one piece of confusion around these methods that each book I've 
read goes about explaining in a slightly different way. None of them 
really clicked for me, so why not write my own?

The two entirely accurate but seemingly paradoxical statements are this:

> Use `class_eval` to define instance methods
>
> Use `instance_eval` to define class methods

The reason for the backwards-ness is often explained something like 
this:

> `x.class_eval` treats `x` as a `Class`, so any methods you create will 
> be instance methods.
>
> `x.intance_eval` treats `x` as an instance, so any methods you create 
> will be class methods.

Well that's clear as mud...

## My take

Here's how I think about it:

> Any methods you define inside of `x.instance_eval` will be as if you 
> had defined them **on** the instance `x`.
>
> Any methods you define inside of `x.class_eval` will be as if you had 
> written it **in** the Class `x`.

Examples should help...

## class_eval

Here's an example of `class_eval`

```ruby 
class MyClass
  def my_method
    "foo"
  end
end

MyClass.class_eval do
  def my_other_method
    "bar"
  end
end

c = MyClass.new
c.my_other_method
=> "bar"
```

This is exactly as if you had done the following:

```ruby 
class MyClass
  def my_method
    "foo"
  end

  # oh... the files are /inside/ the computer!
  def my_other_method
    "bar"
  end
end

c = MyClass.new
c.my_other_method
=> "bar"
```

So we used `class_eval` to define an *instance* method. Just like the 
book said.

Funny thing is, you can easily use `class_eval` to define class methods 
too.

```ruby 
class MyClass
end

MyClass.class_eval do
  def self.foo
    "foo"
  end
end

MyClass.foo
=> "foo"
```

So I think that whole mindset is incorrect. It's about the context your 
code is evaluated in, not what you're intending that matters.

## instance_eval

Similarly, here's how I think when I'm writing something with 
`instance_eval`:

```ruby 
c = MyClass.new

# notice we act *on* an instance
c.instance_eval do
  def my_other_other_method
    "baz"
  end
end

c.my_other_other_method
=> "baz"

# we've written that method *on* c, so it only exists for that 
# *instance*...
d = MyClass.new
d.my_other_other_method
=> Error...
```

This code is identical to

```ruby 
c = MyClass.new

# definition on c
def c.my_other_other_method
  "baz"
end

c.my_other_other_method
=> "baz"
```

In the second form, it's clearer that the method only exists on that 
specific instance.

One other way to look at it is this:

> Methods defined with `class_eval` will be available to every instance 
> of that *class* (making them instance methods).
>
> Methods defined with `instance_eval` will only be available to that 
> specific *instance*; why they're called "class methods", I do not 
> know.

Anyway, hope this helps...
