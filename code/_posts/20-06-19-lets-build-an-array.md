---
title: "Lisp-style Lists in Ruby: A Fun If Not Pointless Exercise"
---

Ruby's arrays are super easy to work with. You just put some stuff between a `[`
and a `]` separated by `,`s and boom you have an array. You can easily do fun
things with your array, iterating like crazy, the world is your oyster. We've
got `each`, `map`, `inject`, `select`, `reject`. All sorts of stuff. Most
languages are like this. It's all easy and very handy and well documented. Which
is why it would be totally pointless to build our own version of `Array` in
Ruby.

## Let's Build Lists Instead

I really like [Lisp-style
lists](http://www.gigamonkeys.com/book/they-called-it-lisp-for-a-reason-list-processing.html).
Elixir uses them. Probably some other languages that I haven't used use them
too. A **list** is similar to an array in that it's a container for objects that
you can iterate over. Lists have some unique properties though:

1. Lists always consist of two elements: a **head** and a **tail**
2. The head of a list can be anything (including other lists)
3. The tail of a list is always itself a list
4. Empty lists are `nil`

That's it. Super simple but also super flexible as far as data structures go.
Plus they're built for functional programming paradigms, as we'll see soon.

## What To Call This Sumbitch

Naming is hard yada yada yada. We'll call our class `Hooray`. That's like
`Array` but more excited and happy to see you.

```ruby
class Hooray
end
```

The first thing we want to do is initialize our `Hooray` in adherence to the
properties of a list as stated above. 

```ruby
class Hooray
  attr_accessor :head, :tail

  def initialize(*args)
    if args.any?
      @head, *tail = args
      @tail = 
        if tail.first.is_a? Hooray
          tail.first
        else
          tail.any? ? Hooray.new(*tail) : Hooray.new
        end
    else
      @head = nil
      @tail = nil  
    end
  end
end
```

We'll give instances of `Hooray` two attributes: `head` and `tail`. That's the
easy part, the initialization needs a bit of explaining. First, we check if
there are any `args` if not we'll just set our `head` and `tail` to `nil`. If
there are `args` we do this:

```ruby
@head, *tail = args
```

This will set `head` to the first `arg` and `tail` as an array containing all
the rest. (Isn't using `Array` stuff cheating? Nope! 😉). After that it does get
a bit tricky but hopefully things will clear up as you keep reading. I'll
explain it anyway. First the `if`:

```ruby
@tail = 
  if tail.first.is_a? Hooray
    tail.first
```

If the tail is already an instance of `Hooray` we'll just use that as the `tail`
as is. In theory you could do something like `Hooray.new(1, Hooray.new(2), 3)`
and that would end up dropping the `3`, but that's ok because rule #1 of lists
is that they only contain two elements. And rule #3 says the second element (our
`tail`) is always another list. Moving on to the `else`:

```ruby
else
  tail.any? ? Hooray.new(*tail) : Hooray.new
end
```

This says if there is anything in the tail, pass them all as arguments to a new
`Hooray` otherwise create a new `Hooray` with no args. This line is responsible
for creating nested `Hoorays` eventually ending in a `nil`, satisfying our
rules.

Now we can fire up `irb`, load our class, and build these things:

```ruby
$ irb
>> load './hooray.rb'
=> true
>> Hooray.new
=> #<Hooray:0x00007f896999f6c0 @head=nil, @tail=nil>
>> Hooray.new(1, 2, 3)
=> #<Hooray:0x00007fdfa89ff800 @head=1, @tail=#<Hooray:0x00007fdfa89ff788 @head=2, @tail=#<Hooray:0x00007fdfa89ff710 @head=3, @tail=#<Hooray:0x00007fdfa89ff698 @head=nil, @tail=nil>>>>
```

This is a pretty good start, basic Ruby stuff, but there are a couple of issues
we need to address out of the gates.

## A Couple Of Issues We Need To Address Out Of The Gates

### First Issue

You may be wondering why our instances of `Hooray` with no args set `head` and
`tail` to `nil` rather than their own `Hooray`s. Two answers there:
1) That would cause an infinite regress of `Hooray`s (which is always
   interesting and funny but not super practical)
2) Keep reading!

### Second Issue

Rule #3 says that "Empty lists are `nil`". With our implementation that's not
entirely true, but it's an easy fix.

```ruby
class Hooray
  # Stuff we've already written

  def nil?
    head.nil? && tail.nil?
  end
end
```

We're defining our own `nil?` method and all it does is check to see if both the
`head` and `tail` are themselves `nil`, if so the `Hooray` itself is `nil`. This
is also our first peek into the simple power of this data structure: since
`tail`s are always lists (in our case instances of `Hooray`) the call to `nil?`
is a call to the method we just wrote. It's recursive. It also handles our no
arg instances of `Hooray` which actually do use `nil` as values. Let's look:

```ruby
>> Hooray.new.nil?
=> true
>> Hooray.new(1).tail.class
=> Hooray
>> Hooray.new(1).tail.nil?
=> true
```

### Third Issue

Look at this mess:
```ruby
>> Hooray.new(1, 2, 3)
=> #<Hooray:0x00007fdfa89ff800 @head=1, @tail=#<Hooray:0x00007fdfa89ff788 @head=2, @tail=#<Hooray:0x00007fdfa89ff710 @head=3, @tail=#<Hooray:0x00007fdfa89ff698 @head=nil, @tail=nil>>>>
```

If we're going to compete with `Array` (we're not) then we need to tidy that up.
Let's add `Hooray#inspect` since `inspect` is used by Ruby to determine how
objects are printed.

```ruby
class Hooray
  # Stuff we've already written

  def inspect
    head_i = "(#{head.inspect}"
    tail_i = tail.nil? ? ")" : tail.inspect.gsub("(", " ")

    head_i + tail_i
  end
end

>> Hooray.new(1, 2, 3)
=> (1 2 3)
```

Much better! Very Lispy!

We should also make these things a little easier to construct. Using
`Hooray.new` isn't terrible, but it doesn't feel like you're constructing a
built-in data type. To fix that we can add a method on `Object` like so. Just
throw it in at the bottom of your `hooray.rb`.

```ruby
class Hooray
  # Stuff we've already written
end

class Object
  def cons(*args)
    Hooray.new(*args)
  end
end
```

In honor of Lisp, we've defined `Object#cons` to make it a tad easier to
construct our lists.

```ruby
>> cons(1, 2, 3)
=> (1 2 3)
```

Ok, I would say we're in a pretty good spot to start doing some actual things
with our lists.

## Doing Some Actual Things With Our Lists

### `#length`
Let's start with something basic, list length. How long is a list? We don't
know, but we teach it how to tell us:

```ruby
def length
  return 0 if nil?

  1 + tail.length
end

>> cons(1, 2, 3).length
=> 3
```

The key here is the operation on `tail`. Since we know that `tail` is always
itself an instance of `Hooray` (rule #3), the call on `tail` is itself the
`Hooray` instance method `length`. This is recursive, making its way all the way
through each `head` of every list. This will be a recurring theme (dad joke)
throughout most of our methods. Even though this isn't a post about recursion it
really kind of is so let's break down what's happening real quick:

```ruby
# We'll use the literal notation without the constructor for simplicity
(1 2 3).length
1 + (2 3).length
1 + 1 + (3).length
# Remember tails are always lists, and empty lists are always nil
1 + 1 + 1 nil.length
1 + 1 + 1 + 0
3
```

### `#==`
We'll say two lists are equal if all the elements have the same values, in the
same order:

```ruby
def ==(hooray)
  head == hooray.head && tail == hooray.tail
end

>> cons(1, 2, 3) == cons(1, 2, 3)
=> true
```

### `#[]`
Another pretty important feature we want is access. With arrays you would use an
index: `[1, 2, 3][2]` to access the third element of the array, `3` in this
case. We'll do something similar for lists, except, being purists, we'll return
the _list_ at the index:

```ruby
def [](i)
  return self if i.zero? || nil?

  tail[i - 1]
end

>> cons(1, 2, 3)[1]
=> (2 3)
```

### `#val`
Sometimes we might actually want what `Array#[]` does and return the head at a
certain index:

```ruby
def val(i)
  return head if i.zero? || nil?

  tail.val(i - 1)
end

>> cons(1, 2, 3).val(1)
=> 2
```

### `#+`
Let's give our lists the ability to join forces. We'll define `+` to append
lists:

```ruby
def +(hooray)
  return hooray if nil?

  Hooray.new(self.head, tail + hooray)
end

>> cons(1, 2) + cons(3, 4) + cons(5, 6)
=> (1 2 3 4 5 6)
``` 

Here we're creating a brand new list from the lists we're smashing together.

## Let's Enumerate!

This is where people get excited about any type of list-like data structure. We
love looping over stuff. Just like above, this is all built on top of recursion,
and it's actually very simple.

### `#each`
The staple, the old trusty, the steadfast tool in your arsenal of looping:
`each`. For each `head` in our list, lets run some code, that's what we're doing
here:

```ruby
def each(&block)
  return if nil?

  yield head

  tail.each(&block)
end

>> cons(1, 2, 3).each { |i| puts "Hi there, #{i}" }
Hi there, 1
Hi there, 2
Hi there, 3
```

Pretty neat right? Thanks to Ruby's closures we can basically get it to act like
a functional language and pass around a block of code (`&block`) to be executed
on each head (`yield head`). That's it. Easy.

With that pattern in place it's a short putt to some other basic enumerable
methods.

### `#map`
Here we want to execute our function on each head and return a _new_ list.

```ruby
def map(&block)
  return Hooray.new if nil?

  Hooray.new((yield head), tail.map(&block))
end

>> cons(1, 2, 3).map { |i| i * i }
=> (1 4 9)
```

### `#reduce`
Reducers are good for taking a list and returning a single value. A common use
case is summing the elements in a list. Our `reduce` method will take an initial
value and a block. 

```ruby
def reduce(value, &block)
  return value if nil?

  tail.reduce(yield(value, head), &block)
end

>> cons(1, 2, 3).reduce(0) { |value, i| value + i }
=> 6
```

Each time through the method will create a new value to pass to the recursive
call to the current `tail` by `yield`ing to the previous `value` and the current
`head`. Reducers can make your head spin a bit (at least they do for me), so let's
step through it:

```ruby
# Again we'll use the literal notation for our lists
(1 2 3).reduce(0) { |0, 1| 0 + 1 } # => 1
(2 3).reduce(1) { |1, 2| 1 + 2 } # => 3
(3).reduce(3) { |3, 3| 3 + 3 } # => 6
nil.reduce(6) => 6
```

### `#mapsum`
We're not going to recreate _every_ `Array` and `Enumerable` method for
`Hooray`, but I wanted to quickly show that these methods can be composed for
great power and profit!

Let's say we want to get the sum of some operation performed on a list of
integers, like, for example the sum of their cubes. Enter `mapsum`!

```ruby
def mapsum(&block)
  map(&block).
  reduce(0) { |x, y| x + y }
end

>> cons(1, 2, 3).mapsum { |i| i ** 3 }
=> 36
```

The main take away here is that these methods can be composable, meaning you can
combine them in different ways to produce new encapsulations.

## Some Things We Didn't Cover
There are a few important and interesting topics that come up when we start
discussing and comparing data structures which we didn't cover because they're
beyond the scope of this post. Each of these could be their own post and are
definitely worth exploring on your own if you're curious.

### Binary Trees
Our `Hooray` lists are really nice for creating [binary
trees](https://en.wikipedia.org/wiki/Binary_tree#:~:text=In%20computer%20science%2C%20a%20binary,child%20and%20the%20right%20child.).
Simply alias `head` to `left` and `tail` to `right` and you got yourself a stew!

### Time Complexity
There are [time complexity](https://www.bigocheatsheet.com/) considerations for
any type of data structure, especially when it comes to accessing, searching,
inserting, and deleting elements. Some are faster for certain operations than
others.

### Runtime
I did't show you any benchmarking, but believe me when I say that our `Hooray`s
run much slower than `Array`s. Is there a way to close that gap?

## Wrapping Up, Finally
That might've seemed like a lot to go through for something that isn't really
practical. But there are a few important takeaways from going through an
exercise like this:

1) You learn some things about Ruby
2) You learn some things about functional programming (go check out
   [Elixir](https://elixir-lang.org/)!)
3) Maybe I sparked your interest in Lisp, a classic and interesting language
   that served as an inspiration for Ruby (go learn
   [Lisp](http://www.gigamonkeys.com/book/)!) 
5) It was fun!

The entire source for `Hooray` can be found
[here](https://gist.github.com/jgoggles/884becd9a9b494af762afeac0b8e6433). I
encourage you to use it and play with it! Can you implement some other array
methods like `select`, `reject`, `slice`? Can you compose some of your own?
