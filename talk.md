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

Descriptions do not change things (unless you're a god), and so neither do
functions in FP. The manifestations of this principal in FP are immutability and
functional purity, which means freedom from side-effects. [0.2, 24]

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


## Ghost 2: Mutability

Descriptivity implies immutability. If I've already said `x = 5`, I can't later
say `x = 2`. What am I, some kind of liar? [11].

You can reassign variables in OO languages because they're about *what to do*,
not what things are. You can say "make x 5", and then "make x 2". But as
descriptions, both cannot be true.

Data in Elixir is immutable. If you're like me then, when you heard this, you
thought that re-assigning variables is forbidden. {reassigning variables
example} {luke and Yoda picture}

This isn't the case. Variables only point to data. You can change these
pointers. Doing so allocates a new location in memory, stores a new value there,
and points the variable to that location. {animated diagram}

This became a theme I noticed. In practice, FP can be very similar to imperative
languages, or at least in languages that aren't necessarily known as FP
languages. This is in part because imperative languages (such as Ruby) have
borrowed ideas from FP. [10, 10b] Thanks to this, you can get your feet wet in
FP, in a language you already use!

- Benefits

With immutability, you can forget your fears of side effects.

It also eliminates the need to copy data, giving performance benefits. If
another variable is assigned to the value of an initial variable, they can both
point to the same place (memory reuse[Cardarella]), safe in the knowledge that
no operation on either variable will result in the changing in the value of the
other. [12]


## Ghost 3: elsif
{pattern matching}

[https://www.quora.com/Why-dont-pure-functional-programming-languages-provide-a-loop-construct/answer/Tikhon-Jelvis]
[http://stackoverflow.com/questions/25067231/why-dont-imperative-languages-have-pattern-matching]

The Getting Started Guide [15] describes `if...else`, but auspiciously without
an `elsif`. Searching the page for `else if`, I learned that `cond` is the
closest thing to `else if` in Elixir.

Elixir has the `cond` control structure, which
the Getting Started guide says is "This is equivalent to else if clauses in many
imperative languages (although used way less frequently here)." [15]

```elixir
  def find_top(col, n\\0) do
    col_list = Tuple.to_list(col)
    [h|t] = col_list
    
    cond do
      !Enum.any?(t) && h != 0 ->
        {:error}
      h == 0 ->
        {:ok, n}
      true ->
        List.to_tuple(t) |> find_top(n+1)
    end
  end
```

Here's an example from Connect 4 in Elixir

- Pattern Matching

Although there is an equivalent control structure, I still found it odd. Why
would a language eschew such seemingly fundamental control structures? I thought
there had to be more to it. Again, the common theme is the principle of writing
code that's easier to understand or "reason about".

Elixir gives us the tool of _pattern matching_. Aside: pattern matching is not
(afaik) among the core principles of FP. But I feel that it follows in the
spirit of FP so faithfully that its worth talking about. Also, it was too late
for me to go back and change the abstract for my talk.

```elixir
  [h|t] = col_list
```

We saw pattern matching in the previous example. The pipe here is known as a
"cons" operator, and it represents link between the two elements. In Elixir,
Lists are implemented as linked lists; each node consisting of a value and a
pointer to the next node. The `cons` represents that link to the next node. If
this expression represents a match between the left side and right side, then
`h` must be the first element of `col_list`, and  `t` the remainder. (The whole
thing is called a *cons cell*)
[http://elixir-lang.org/getting-started/basic-types.html]

But pattern matching plays a much bigger role in Elixir. The most exciting
(to me) is in function definitions! Let's see how pattern matching
is used here.


```elixir
  def check(remaining, []) do
    [next_row | other_rows] = remaining
    check(other_rows, next_row)
  end
  
  def check([], row) do
    check_row(row)
  end

  def check(remaining, row) do
    if check_row(row) do
      true
    else
      [next_row | other_rows] = remaining
      check(other_rows, next_row)
    end
  end
```

The `check` function has multiple clauses, each with a different signature. The
precedence in matching is the order of appearance; so if a clause matches, no
clauses below it will be attempted.

The first clause applies if the second argument is an empty list, the second
clause if the first argument is an empty list, and the last clause if neither
argument is an empty list.

This accomplishes the same effect as control-flow structures, but is easier to
read. Let's have a look at the equivalent code implemented with control-flow
structures for comparison.

```elixir
  def check(remaining, row) do
    if Enum.empty?(row) do
      [next_row | other_rows] = remaining
      check(other_rows, next_row)
    else
      if !Enum.empty?(remaining) do
        check_row(row)
      else
        if check_row(row) do
          true
        else
          [next_row | other_rows] = remaining
          check(other_rows, next_row)
        end
      end
    end
  end
```


`elsif` enables one to create spaghetti code. With it, a single function can
house logic for many different scenarios. Granted, you can achieve the same
result by nesting another `if...else` in the `else` of the original `if...else`,
but I think anyone faced with the monstrosity that would result will be
compelled to look for a better way. (Either that or they'll storm off saying
"Elixir is stupid!".)


- Inversion of Control

An article I read [19] helped give me more perspective about why one might want
to avoid conditionals. It is entitled "Destroy All Ifs — A Perspective from
Functional Programming".

John explains that in imperative languages, when you call a function, you give
it control. In functional languages, control is _inverted to the caller_; a
function does nothing except return a value (functional purity). The calling
function may use or ignore this value. He summarizes this by saying:

>Code that inverts control to the caller is generally easier to understand.
[19]


## Ghost 4: Loops

- Loops

Immutability is great, but it has some profound implications on the way we
program. Let's take a look at one example. From discussions with Elixir users,
and reading the getting started guide [15] and some articles [4, 9, 9b], I
gathered that recursion was important and assumed that there were no loops in
Elixir. 


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

```ruby
while i_am_talking do
  be_nervous
end
```

The loops we typically use in OO programming rely on mutability. Assuming the
code inside the block is "pure", then it would evaluate the same way every time.
That means either it will return on the first iteration, or it will never
return. The former is indistinguishable from having no loop, and the latter is
an infinite loop, which is usually not something we want.

Thinking that there were not loops in FP, I spent time practicing solving
problems with recursion.

- Recursion

In FP, loops are replaced by recursive functions!

What is recursion?!

```elixir
  def fibonacci(n) do
    cond do
      n == 0
        0
      n == 1
        1
      true ->
        fibonacci(n, 0, 1)
    end
  end

  def fibonacci(n, a, b) do
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

- Example

```ruby
def self.check(grid)
  grid.each_with_index do |column, column_index|
    n = 1
    last = 0
    column.each_with_index do |cell, row_index|
      if cell == last && cell != 0
        n += 1
      else
        n = 1
      end
      last = cell
      
      return true if n == 4
    end
  end
```

Here's how we check the Connect 4 grid for horizontal matches in Ruby; its
extremely standard. We just iterate over the rows, and over the cells in each
row, keeping track of consecutive same-colored pieces.


```elixir
  def check_horizontal(board) do
    Transpose.transpose(board)
    |> check([])
  end

  def check(remaining, []) do
    [next_row | other_rows] = remaining
    check(other_rows, next_row)
  end
  
  def check([], row) do
    check_row(row)
  end

  def check(remaining, row) do
    if check_row(row) do
      true
    else
      [next_row | other_rows] = remaining
      check(other_rows, next_row)
    end
  end
```

You've already seen the corresponding Elixir code; I showed it when talking
about pattern matching. This code uses recursion to check every row of the board
(which is a 2D array) for four of a players pieces in a row.

`check_horizontal` calls `check`, passing the board as the first argument and
an empty list as the second. That means the first clause of `check` will be applied.
There, we use pattern matching again to pull off the next row from the board.
The remainder of the board is passed as arg 1, and the single row passed as arg 2.

Note, we could also use pattern matching in the argument list to simplify this function:

```elixir
  def check([next_row | other_rows], []) do
    check(other_rows, next_row)
  end
  
  # Equivalent to:
  def check(remaining, []) do
    [next_row | other_rows] = remaining
    check(other_rows, next_row)
  end
```

With both arguments holding something, the third clause takes effect. Here, we
check the single row for a win. If none is found, then just like the first
clause, we peel off the next row from the board and pass the board and it
separately to `check`.

```elixir
[next_row | other_rows] = remaining
```

Eventually, the board will be reduced to a single row. In this case,
`other_rows` will be an empty list. The second clause will finally be in charge.
There, all that's left to do is check that remaining row. If there are no wins
there, then we can just return false.


- Tail Recursion

{ Video: You've got one on your tail! }

I want to briefly mention _tail-recursion_. This means that a function that
calls itself, does so as the last subroutine. The compiler is able to call the
function without creating a new stack frame.


- There are "loops"!

After becoming a recursion master, I found out that "there aren't loops in FP"
isn't true! [14] At least not quite. There are some constructs in Elixir that
behave uncannily like loops, including algorithms in the `Enum` module, and list
comprehensions.

Here is another instance of the trend that FP ideas are present in other
languages. Ruby, what used to be my favorite language, has `map`, `reduce`, and
`select`. These are known as higher-order functions; that is, functions that
accept or return other functions.

```elixir
Enum.reduce( (1..100), fn(x, acc) -> x + acc end )
  # => 5050

Enum.map( (1..5), fn(x) -> 2 * x end )
  # => [2, 4, 6, 8, 10]

Enum.filter( (1..5), fn(x) -> rem(x, 2) == 0 end )
  # => [2, 4]
```

Examples of these in use in Elixir.

They are not technically loops; they are implemented differently from those in
imperative languages I don't know the details, but the main difference is
immutability. It seems like much of the difference between the imperative
languages _I'm_ used to and FP is under the hood.

I was pleased to learn that I've been using FP principles all along, but I was
also a tiny bit bummed, because it felt like that would leave me less to talk
about. Eventually I realized that I shouldn't be upset that something is easier
to learn that I expected. This means that more people can get into Elixir and
Phoenix, and take advantage of their strengths (performance, scalability), with
less work. That's a good thing!
