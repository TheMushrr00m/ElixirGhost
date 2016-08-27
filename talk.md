## Intro

Hi, I'm Morgan Laco. I'm a software engineer at Code School, which is a company
that teaches web technologies in the comfort of your browser with video lessons,
coding challenges, and screencasts. We strive to help you learn by doing. I
primarily use Ruby on Rails.

I was introduced to Elixir at Brian Cardarella's talk at RailsConf, which was
fantastic, btw. Please watch it on ConFreaks.

This talk is billed for OO / imperative devs, but really its for any dev who is
new to FP. However, if you're a dev, but not a FP dev, you're probably an OO
dev. This talk assumes a basic level of programming compentency.

- Context 4!

For context, I'll use a Connect 4 app. A brief history of that app...  It
started when Sergio Cruz was writing a React app
[https://github.com/sergiocruz/react-connect4] for a contest. I wrote logic
checking for diagonal matches on the board. I was able to reduce the code from a
triple nested loop to a double-nested loop. {Diagram?}
[https://github.com/sergiocruz/react-connect4/blob/master/app/components/connect-4/lib/matches/diagonal.js]

I decided to rewrite this app in Elixir to provide the context of this talk.
Before I wrote the Elixir app, I also wrote it in Ruby because I felt it would
be better to compare Elixir to Ruby since the syntax is so similar. This way
there would be fewer differences in syntax to obscure the differences we're
really interested in.

- Making the Switch

Much of the difference from OO that I noticed, I noticed because of something
missing, such as control structures or statements, mainly when going through
the Elixir Getting Started Guide [15].

I've based my talk on the things I found missing... these "ghosts" of imperative
programming. I'll discuss what replaces them, and the philosophy behind the
replacement, and that will hopefully help you give up these ghosts. The important
thing is not the ghost, but "what killed it", or in other words, the _reason_
why the familiar thing is missing.


## What is Functional Programming?

If you're here, then you probably have some idea about this already. But maybe
you've just gotten into Elixir or FP in general, so I'll briefly discuss what FP
is all about.

- Backstory!

{ T-4a Lambda-class shuttle }

Functional programming comes from lambda calculus,
which is a formal system invented by Alonzo Church in the 1930s as part of an
investigation into the foundations of mathematics.

[https://en.wikipedia.org/wiki/Lambda_calculus]
[A. Church, "A set of postulates for the foundation of logic", Annals of Mathematics, Series 2, 33:346–366 (1932).]


Lambda calculus was developed to achieve a clearer approach to the foundation of
mathematics. Although a first look at LC expressions, you would think it makes
things much less complicated (e.g. Church numerals) { Church numeral example }

0 := 𝛌f.𝛌x.x
1 := 𝛌f.𝛌x.f x
2 := 𝛌f.𝛌x.f (f x)
3 := 𝛌f.𝛌x.f (f (f x))

[Haskell Brooks Curry; Robert Feys (1958). Combinatory Logic. North-Holland Publishing Company. Retrieved 10 February 2013.
via https://en.wikipedia.org/wiki/Lambda_calculus]

In the same way, functional programming is meant to be a clearer approach to
programming. This is achieved by treating functions (i.e. subroutines [3]) as
descriptions of things rather than instructions [6].

Descriptions do not change things, and so neither do functions in FP. The
manifestations of this principal in FP are immutability and functional purity, which
means freedom from side-effects. [0.2, 24]

In lambda calculus, functions are mathematical (though anonymous) objects.
Similarly in FP, functions are first-class citizens. They can be passed around
as you would an integer, for instance [6]. This isn't too uncommon though;
functions are first class objects in JS too.

Something to keep in mind. Functional programming, like other paradigms, can be
seen as a bunch of habits that we apply during programming. [10] Also, this talk is
about FP, rather than Elixir specifically. 


## Ghost 1: Objects

Perhaps the most obvious difference from OO is that FP has no objects. {example -
game is not an object}

The philosophy behind objects is to represent the real world; objects have
attributes, data associated with them, and ways to use them or change those
attributes in specific ways, "methods". It is obviously antithetical to the
tenets of FP to change those that data.

```ruby
class ConnectFour::Game
  def initialize
    @board = ConnectFour::Board.new
    # other instance variables
  end
  
  def play
    while continue_game
      get_move
      handle_move
    end
    
    display_result
  end
  # ...
end

ConnectFour::Game.new.play
```

```elixir
defmodule ConnectFour do
  def start(_type, _args) do
    ConnectFour.initial_state
    |> do_turn
  end
  
  def do_turn(state) do
    ConnectFour.IO.display_board(state[:board])
    
    case get_new_state() do
      # ...
      new_state ->
        if ConnectFour.WinChecker.check(new_state[:board]) do
          ConnectFour.IO.do_win(new_state)
        else  
          do_turn(new_state)
        end
    end
  end

ConnectFour.start
```

{Note: this is essentially pseudocode, it has been modified from the original}

If you aren't familiar with the pipe operator `|>`, it is Elixir syntax which
takes the result of the previous function and passes it as the first argument
to the next function.

(In the code,) we see the difference this makes. In imperative code, we create
the `ConnectFour::Game` object, which holds instance variables, most importantly
`@board`, though there are others in the actual code.

In the imperative code, the game object's data can be freely modified anywhere
else where it is in scope. Any code can just swoop down and change things.

Contrast this with the functional code. We are required to pass the board state
along. Remember, that's what `|>` does. The `start` function is equivalent to
this:

```elixir
def start(_type, _args) do
  state = initial_state
  do_turn(state)
end
```

`do_turn` is the function that really does all of the work, and you can see here
too that we're constantly passing the board state along. It looks a little less
clean, but it is fully explicit. You don't have to hold the state in your mind,
because its right there in the file.




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
