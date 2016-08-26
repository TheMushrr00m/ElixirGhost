## Intro

Hi, I'm Morgan Laco. I'm a software engineer at Code School, which is a company
that teaches web technologies in the comfort of your browser with video lessons,
coding challenges, and screencasts. We strive to help you learn by doing. I
primarily use Ruby on Rails.

This talk is billed for OO / imperative devs, but really its for any dev who is
new to FP. However, if you're a dev, but not a FP dev, you're probably an OO
dev. This talk assumes a basic level of programming compentency.


## What is Functional Programming?

If you're here, you probably have some idea about this already. But maybe you've
just gotten into Elixir, so I'll just briefly cover this.

FP has its roots in Lambda Calculus in the 1930s

Core ideas of FP:
- lambda calc
- Higher order functions
- Immutability
- No side-effects
[24]

Functions are ordinary mathematical objects, just like an integer[6]. This isn't
too uncommon though; functions are first class objects in JS too.

It is declarative rather than imperative. It consists of *descriptions* of
things rather than *instructions* for what to do with things. [6]

That perspective is that the spirit of FP is about putting to work powerful
logical, mathematical concepts that have languished in relative obscurity due to
their opacity. Concepts that can be difficult to understand, that are rather
abstract and "theoretical", and because of this they do not enjoy widespread
understanding.

Practically, FP is about one thing: absence of side effects [0.2]



## Making the Switch

Much of the difference from OO that I noticed, I noticed because of something
missing, such as control structures or statements, mainly when going through
the Elixir Getting Started Guide [15].

{Talk about Connect 4}


## No Objects

My goal is to help you make the switch from OO to FP. What will help you in that
process is learning the differences between the two. The differences are in both
the structure of the language itself, and in the way you use it to solve
problems. It is more important, and probably more difficult, to get used to the
latter.

The most obvious difference from OO is that FP has no objects. {example - game
is not an object}

This means you can't chain method calls the way we can in Ruby, for instance.

So what do we do? Of course we could nest function calls, but that's not fun.

We can use the *pipe* for these situations. {example of pipe usage}

Note that you can't assign the result of a chain of pipes to a variable. This is
actually a good thing.

In this situation, the entire chain should be a separate function. This
encourages clean, modular code. {example} There are several ways FP encourages
good code quality.


## Fewer Control Structures

The Getting Started Guide [15] describes `if...else`, but auspiciously without
an `elsif`. Searching the page for `else if`, I learned that `cond` is the
closest thing to `else if` in Elixir. But I'm not here to walk you through the
Getting Started guide. I'm here to philosophize about what I learned!

The interesting question is why would a language eschew such seemingly
fundamental control structures? Some articles I read later [16, 19] helped give
me perspective on this.

But sometimes seemingly complicated things become simpler when they're hands-on,
or when someone else understands them a bit and explains them to you in a more
practical way. Elixir is doing that.

Actually, concepts from FP have been making their way into imperative langauges for
some time. Lambda expressions are present in Ruby, Java since Java 8 [1] Strings
are immutable in Python [0.2]

> Libraries for employing monadic composition to chain I/O processing without
blocking kernel threads have sprung up in imperative languages – the beautiful
ReactiveX (C#, Java, JS), Node.js Promises (JS), Facebook's C++ Futures – and
monadic composition has even found itself in the JDK itself, with the
introduction of CompletionStage (and CompletableFuture which implements it) in
Java 8.

[16]

Monads allow us to avoid callback hell!


## Immutability

Descriptivity implies immutability. If I've already said `x = 5`, I can't later
say `x = 2`. What am I, some kind of liar? [11].

You can reassign variables in OO languages because they're about *what to to*,
not what things are. You can say "make x 5", and then "make x 2". But as
descriptions, both cannot be true.

Data in Elixir is immutable. If you're like me then, when you heard this, you
thought that re-assigning variables is forbidden. {reassigning variables
example} {luke and Yoda picture}

This isn't the case. Variables only point to data. You can change these
pointers. Doing so allocates a new location in memory, stores a new value there,
and points the variable to that location. {animated diagram} [How is this
different from OO? Can you change the data in-place in OO?]

- Benefits

With immutability, you can forget your fears of side effects.

It also allows eliminates the need to copy data. If another variable is assigned
to the value of an initial variable, they can both point to the same place, safe
in the knowledge that no operation on either variable will result in the changing
in the value of the other. [12]


## Loops -> Recursion

- Loops

Immutability can be great, but it has some profound implications on the way we
program. Let's take a look at one example.

```ruby
(1..100).each do |i|
  do_something(i)
end
```

Loops such as this inherently depend on mutability. Here, the value of `i`
changes in each iteration. So, you won't see the usual basic loops (`for`,
`while`) in Elixir. Here's a look at Elixir's Getting Started guide; [15]

We have seen that variables can be redefined; it is only _data_ that is
immutable, not the value of variables.

So why can't we have loops like this? On each iteration, we would just create
some new data, and point `i` to that data. It goes against the paradigm of
modularity / purity.

The loops we typically use in OO programming rely on mutability. Assuming the
code inside the block is "pure", then it would evaluate the same way every time.
That means either it will return on the first iteration, or it will never
return. The former is indistinguishable from having no loop, and the latter is
an infinite loop, which is usually not something we want.

```ruby
while i_am_talking do
  be_nervous
end
```

```ruby
begin
  be_nervous
end while i_am_talking
```

So after I read the getting started guide [15] and some articles [4, 9] and had
conversations with Elixirans, I gathered that recursion was important and
assumed that there were no loops in Elixir. It turns out this isn't true! [14]
However, I did spend some time practicing recursion, along with the other tools
of FP/Elixir.

{Example of recursion (from Connect 4?)}

- Recursion

In FP, loops are replaced by recursive functions!

What is recursion?!

{ Fibonacci example }
```elixir
# n - the ordinal number of the term in the sequence desired
# a, b - internal use; keeps track of last two terms
def fibonacci(n, a \\ 0, b \\ 1) do
  if n <= 0 do
    a
  else
    fibonacci(n - 1, b, a + b)
  end
end
```

Recursion is when a function calls itself. The cycle of the function calling
itself must be broken by a base case, in which the function determines the
result without calling itself. In the above example, we can determine the result
when we have reached the term with the ordinal number (its position in the
sequence) that we want.

[Note: in this example, I'm using default arguments this way because we haven't
gotten to pattern matching.]

- Tail Recursion

{ Video: You've got one on your tail! }