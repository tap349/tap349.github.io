---
layout: post
title: Elixir notes
date: 2016-12-31 13:05:42 +0300
access: public
categories: [elixir]
---

Elixir notes (mostly based on "Programming Elixir" by Dave Thomas).

<!-- more -->

* TOC
{:toc}

## IEx

- Erlang shell - Eshell (`erl`)
- Elixir shell - IEx (`iex`)
- UNIX shell - Bash, etc.

### shell history

<http://nithinbekal.com/posts/elixir-shell-history/>:

```sh
$ git clone https://github.com/ferd/erlang-history.git
$ cd erlang-history
$ sudo make install
```

shell history since from now is stored in _~/.erlang-hist.nonode@nohost_
(it's a binary file - not plain text).

NOTE: you need to compile it for each new version of Erlang/OTP kernel!

- get current version of Kernel library:

  ```sh
  $ erl
  1> application:which_applications().
  [{stdlib,"ERTS  CXC 138 10","3.2"},
  {kernel,"ERTS  CXC 138 10","5.1.1"}]
  ```
  current version is 5.1.1.

- update `erlang-history` repo:

  ```sh
  $ cd erlang-history
  $ git up
  ```

- make and install:

  ```sh
  $ cd erlang-history
  $ sudo make install
  Password:
  erl -make
  Recompile: src/5.1.1/group_history
  Recompile: src/5.1.1/group
  ...
  ebin/5.1.1/group.beam
  ebin/5.1.1/group_history.beam
  2 file(s) copied
  ```

  make sure `2 file(s) are copied` - if no files are copied it means
  current version of Kernel library is not supported and you should
  probably update `erlang-history` repo.

### evaluate vs. compile in memory vs. compile

<https://github.com/elixir-lang/elixir/issues/5073>

> `ex` files are meant to be compiled<br>
> `exs` files are used for scripting

each module is compiled into its own bytecode (`beam`) file:

- if file doesn't contain modules no bytecode files are generated
- it doesn't matter if module is defined in `exs` or `ex` file<br>
  (file extension doesn't matter at all - module can be defined in `rb` file)
- module `Example` is compiled into `Elixir.Example.beam` file

#### evaluate

- code typed in IEx is always evaluated - not compiled
- code is evaluated iff when typed in IEx directly
- never benchmark by typing code in IEx directly

#### compile in memory

in these cases bytecode modules are not written to disk - only loaded in memory:

- `$ iex test.exs`

  compiles file in memory and loads it into IEx.

- `$ elixir file.exs`

  compiles file in memory and executes it
  (it's like running any other script in UNIX shell).

- `iex> import_file "test.exs"`

  > evaluates the contents of the file as if it were directly typed into IEx

  in spite of what is said above imported file is compiled in memory.

#### compile

- `$ elixirc file.exs`

  compiles file and executes it.

- `iex> c "test.exs"`

  compiles file and loads it into IEx.

## types

- value types:
  - integers
  - floats
  - atoms (booleans are just atoms)
  - ranges
  - regular expressions
- system types:
  - PIDs and ports
  - references
- collection types:
  - tuples
  - lists
  - maps
  - binaries

## pattern matching

during pattern matching all unbound variables must be on LHS of an equals sign:

```sh
iex> a = 1
1
iex> 2 = b
** (CompileError) iex:2: undefined function b/0
```

pattern matching is recursive - we can match patterns inside patterns:

```elixir
def for_location([h = [_, target_loc, _, _] | t], target_loc) do
  [h | for_location(t, target_loc)]
end
```

if `target_loc` argument matches the 2nd element of list head,
`h` variable matches the whole list head (is getting bound to it).

## functions

function consists of head and body:

<dl>
  <dt>head</dt>
  <dd>function name, parameter list, optional guard clause</dd>

  <dt>body</dt>
  <dd>sequence of expressions</dd>
</dl>

### function capturing (& notation aka capture syntax)

`&` - function capture operator

- create anonymous function using provided expression

  ```elixir
  add_one = &(&1 + 1) # form with parentheses
  add_one = & &1 + 1 # form without parentheses
  to_tuple = & {&1, &2}
  ```

  anonymous functions defined using `&` operator must have at least
  one argument (placeholders &1, &2, etc.):

  ```sh
  iex> a = &(1)
  ** (CompileError) iex:29: invalid args for &, expected an expression in the
  format of &Mod.fun/arity, &local/arity or a capture containing at least one
  argument as &1, got: {1}
  iex> a = &(&1)
  #Function<6.52032458/1 in :erl_eval.expr/5>
  iex> a.(1)
  1
  ```

- get anonymous function that calls existing named function

  ```elixir
  puts = &IO.puts/1
  puts.("Hello world")
  ```

  Elixir Guide:

  > this notation can actually be used to retrieve a named function
  > as a function type

  "Programming Elixir":

  > you can give it (& operator) the name and arity of an existing
  > function, and it will return an anonymous function that calls it

  to my mind it's like getting reference to existing named function.

  ```sh
  iex> puts = &IO.puts/1
  &IO.puts/1
  iex> puts.(123)
  123
  :ok
  ```

  it's useful when you want to pass named function
  (either standard or your own one) as another function argument:

  ```elixir
  Enum.each [1, 2, 3, 4], &IO.inspect/1
  ```

- get partially applied named function (currying)

  - <http://blog.patrikstorm.com/function-currying-in-elixir>
  - <https://github.com/Qqwy/elixir_currying>

  it's possible to apply named function partially and pass it as another
  function argument (named function becomes anonymous one in this case).
  unlike real currying you cannot further apply more arguments - only
  once when passing partially applied named function to another function.

  ```sh
  iex> defmodule Example do
  ...>   def pad_leading word, padding do
  ...>     String.pad_leading(word, String.length(word) + padding)
  ...>   end
  ...> end
  {:module, Example, ..., {:pad_leading, 2}}
  iex> partially_applied_fun = &Example.pad_leading(&1, 3)
  #Function<6.52032458/1 in :erl_eval.expr/5>
  iex> partially_applied_fun.("foo")
  "   foo"
  iex> partially_applied_fun = &Example.pad_leading("foo", &1)
  #Function<6.52032458/1 in :erl_eval.expr/5>
  iex> partially_applied_fun.(4)
  "    foo"
  ```

### do...end block

`do...end` block is a way to group expressions treating them as a single entity.

- actual syntax:

  ```elixir
  # enclose multiline do...end block in ()
  defmodule Times, do: (
    # single line do...end block
    def double(n), do: n * 2
    def triple(n), do: n * 3
  )
  ```

  NOTE: if using actual syntax function parameters MUST BE
        enclosed in parentheses!

- with syntactic sugar:

  ```elixir
  defmodule Times do
    def double n do
      n * 2
    end

    def triple do
      n * 3
    end
  end
  ```

### multiple clauses

both anonymous and named functions can have multiple clauses - these are
not multiple function definitions but multiple clauses of the same function
definition (I assume 'function clause' == 'function definition clause').

- anonymous functions:

  ```elixir
  fn
    parameter-list -> body
    parameter-list -> body
  end

  # or:

  fn
    parameter-list ->
      body
    parameter-list ->
      body
  end
  ```

- named functions:

  ```elixir
  defmodule Example do
    def name(parameter-list), do: body
    def name(parameter-list), do: body
  end
  ```

NOTE:

- multiple clauses of named functions must be adjacent (grouped together)
  in the source file
- all clauses of both anonymous and named functions must have the same arity

### private functions

naming convention for private functions with the same name as public functions
(such private functions usually have higher arity than public functions -
e.g. to pass accumulator around):

- prefix with `_` ("Programming Elixir")
- prefix with `do_` ("The Elixir Style Guide")

```elixir
def sum(list), do: _sum(list, 0)

# Programming Elixir
defp _sum([], total), do: total
defp _sum([h | t], total), do: _sum(t, h + total)

# The Elixir Style Guide
defp do_sum([], total), do: total
defp do_sum([h | t], total), do: do_sum(t, h + total)
```

## modules

all named functions must be defined inside modules!

module names are just atoms: any name (not necessarily module name) starting with
an uppercase letter is converted internally into an atom prefixed with `Elixir.`:

```sh
iex> is_atom IO
true
iex> to_string IO
"Elixir.IO"
iex> to_string Example
"Elixir.Example"
iex> :"Elixir.Example" == Example
true
```

NOTE: `Example` module is not even defined - it's just an arbitrary name here.

=> it's possible to call any module function this way:

```sh
iex> :"Elixir.IO".puts 123
```

in Erlang atoms are lowercase names, all module names are atoms
=> to call function from Erlang module in Elixir just convert
Erlang module name into valid Elixir atom:

```sh
iex> :io.format("number is ~3.1f~n", [5.123])
```

in Erlang it's equivalent to:

```erlang
1> io:format("number is ~3.1f~n", [5.123]).
```

### nested modules

"Programming Elixir":

> Module nesting is an illusion - all modules are defined at the top level.
> When we define a module inside another, Elixir simply prepends the outer
> module name to the inner module name, putting a dot between the two.

=> there is no particular relationship between, say, modules
`Google.Adwords.Importer` and `Google`!

### module directives

- <http://elixir-lang.org/getting-started/alias-require-and-import.html>


Elixir has 3 directives for modules, all of them are lexically scoped -
they are effective from the point they are encountered till the end of
enclosing scope (say, function).

- `import`

  use module functions without specifying module name:

  ```elixir
  defmodule Example do
    def func do
      import List, only: [flatten: 1]
      flatten [1, [2, 3], 4]
    end
  end
  ```

  tips:

  - use in the smallest possible scope
  - use `only:` to import only the functions you need

  > Note that importing a module automatically requires it.

- `alias`

  create alias for module (to cut down on typing):

  ```elixir
  defmodule Example do
    def func date do
      alias Google.Adwords.Importer, as: Importer
      date |> Importer.import()
    end
  end
  ```

  other usage examples:

  ```elixir
  # same as above
  # (last part of module name is used by default)
  alias Google.Adwords.Importer
  # alias multiple modules at once
  alias Google.Adwords.{Importer, Parser}
  ```

- `require`

  make macros from specified module available in containing module
  (= ensure `required` module is compiled and available).

  > In general a module does not need to be required before usage,
  > except if we want to use the macros available in that module.


also Elixir has macro `use` (it's not directive) which:

- requires given module
- calls `__using/1__` callback on it (to inject some code into current context)

that is

```elixir
defmodule Example do
  use Feature, option: :value
end
```

is compiled to:

```elixir
defmodule Example do
  require Feature
  Feature.__using__(option: :value)
end
```

### module attributes

```elixir
defmodule MyList do
  @z_ascii_code 122

  def caeser([h | t], n) when h <= @z_ascii_code + n do
    [h + n | caeser(t, n)]
  end
end
```

used:

- for configuration and metadata (like `dsl_attribute` in our projects)
- instead of constants in other programming languages

## lists

```sh
iex> [1 | [2 | [3 | []]]]
[1, 2, 3]
```

### keyword lists

`Keyword` module is used to manipulate keyword lists.

keyword list is a list of tuples where the 1st element is atom:

```elixir
[a: 1, b: 2]
# is equivalent to
[{:a, 1}, {:b, 2}]
```

keyword lists are usually used to store options passed to functions:

```elixir
def draw_text(text, options \\ []) do
  options = Keyword.merge(@defaults, options)
  ...
end
```

### character lists

character list (or char list) is just a list of integers (codepoints) internally -
in IEx it's printed as a list of characters if all characters are printable.

force print char list as collection of codepoints:

- `:io.format/2`

  `~w` forces 'cat' to be written as an Erlang term:

  ```sh
  iex> :io.format "~w~n", ['cat']
  [99,97,116]
  :ok
  ```

- `List.to_tuple/1`

  ```sh
  iex> List.to_tuple 'cat'
  {99, 97, 116}
  ```

- add non printable character codepoint (say, `0`)

  ```sh
  iex> 'z' ++ [0]
  [122, 0]
  ```

- ? notation to get codepoint of single character

  ```sh
  iex> ?c
  99
  iex> ?\s
  32
  ```

## maps

`Map` module is used to manipulate maps.

choosing between maps and keyword lists (both are dictionaries):

|                                 | map   | keyword list |
|:--------------------------------|:-----:|:------------:|
| use in pattern matching         | ✓     |              |
| allow duplicate keys            |       | ✓            |
| preserve order of elements      |       | ✓            |
| use to store options            |       | ✓            |
| use as general key/value store  | ✓     |              |

### pattern matching with maps

- empty map matches any map (empty map must be on LHS)

  ```sh
  iex> %{} = %{1 => :ok, 2 => :error}
  %{1 => :ok, 2 => :error}
  iex> %{1 => :ok, 2 => :error} = %{}
  ** (MatchError) no match of right hand side value: %{}
  ```

- pattern matching cannot bind values to keys

  ```sh
  iex> %{a => :ok} = %{1 => :ok, 2 => :error}
  ** (CompileError) iex:27: illegal use of variable a inside map key match,
  maps can only match on existing variable by using ^a
  ```

  use `^` pin operator for keys to explicitly prohibit binding
  (= use current variable value and don't even try to rebind variable):

  ```sh
  iex> a = 1
  1
  iex> %{^a => :ok} = %{1 => :ok, 2 => :error}
  %{1 => :ok, 2 => :error}
  ```

### structs

struct is kind of a named map with associated behaviour which is
defined in a module by using `defstruct` macro inside it
(internally it's a bare map with `__struct__` key that holds
the name of the struct).

```elixir
# User is a name of the struct
defmodule User do
  # keys with default values (keyword list)
  defstruct name: "", email: "", is_admin: false
  # keys without default values (list of atoms) -
  # nil is assumed
  #defstruct [:name, :email, :is_admin]

  def admin?(%User{name: name, is_admin: is_admin})
      when name != "",
      do: is_admin
end
```

NOTE: [access syntax](#access-module) cannot be used to access struct fields
(only field-based lookup):

```sh
iex> user = %User{}
iex> user[:name]
** (UndefinedFunctionError) function User.fetch/2 is undefined
(User does not implement the Access behaviour)
iex> user.name
nil
```

### nested accessors (from `Kernel` module)

<https://hexdocs.pm/elixir/Kernel.html>

they work with dictionaries (maps and keyword lists):

|                   | macro                    | function            |
|                   | (static accessors)       | (dynamic accessors) |
|-------------------|--------------------------|---------------------|
| get_in            | no                       | (dict, keys)        |
| put_in            | (path, value)            | (dict, keys, value) |
| update_in         | (path, fn)               | (dict, keys, fn)    |
| get_and_update_in | (path, fn)               | (dict, keys, fn)    |

macros are more concise but functions allow to determine the set of
keys at runtime => hence macros are static nested accessors while
functions are dynamic ones.

path in macros is extracted using, well, macro this way:

```elixir
# using access syntax (maps and keyword lists - but not structs)
put_in(opts[:foo][:bar], :baz)
# using field-based lookup (maps only - including structs)
put_in(opts.foo.bar, :baz)
# both are equivalent to
put_in(opts, [:foo, :bar], :baz)
```

BTW `get_in(opts, [:foo, :bar])` is equal to `opts.dig(:foo, :bar)` in Ruby.

#### `Access` module

`Access` module provides functions to be used with `get_in` and
`get_and_update_in` functions to filter elements in lists and tuples or
keys in dictionaries.

- <https://dockyard.com/blog/2016/02/01/elixir-best-practices-deeply-nested-maps>
- <https://github.com/elixir-lang/elixir/blob/v1.2.2/lib/elixir/lib/access.ex#L50>

> `foo[bar]` - access syntax (named after `Access` module where it's implemented)
> `foo.bar` - field-based lookup (aka path form)

functions for lists:

- `Access.all/0`
- `Access.at/1`

```elixir
opts = [%{foo: :bar}, %{foo: :baz}]
# get all :foo values of the list opts
get_in(opts, [Access.all(), :foo]) # [:bar, :baz]
# get the 1st :foo value of the list opts
get_in(opts, [Access.at(0), :foo]) # :bar
```

functions for tuples:

- `Access.elem/1`

```elixir
opts = [%{foo: {1, 2}}, %{foo: {3, 4}}]
get_in(opts, [Access.all(), :foo, Access.elem(1)]) # [2, 4]
```

functions for dictionaries:

- `Access.key/2` (provide default value for missing key)
- `Access.key!/1` (expects key to be always available)
- `Access.pop/1`

```elixir
opts = %{foo: 1, bar: %{baz: 2}}
# this is equivalent to opts.bar.baz:
get_in(opts, [Access.key(:bar, nil), :baz]) # 2
Access.pop(opts, :foo) # {1, %{bar: %{baz: 2}}}
```

## sets

```sh
iex> set_1 = MapSet.new [1, 2, 3]
iex> set_2 = 3..5 |> Enum.into(MapSet.new)
iex> MapSet.member? set_1, 1
true
iex> MapSet.union set_1, set_2
#MapSet<[1, 2, 3, 4, 5]>
iex> MapSet.difference set_1, set_2
#MapSet<[1, 2]>
iex> MapSet.difference set_2, set_1
#MapSet<[4, 5]>
iex> MapSet.intersection set_1, set_2
#MapSet<[3]>
```

## streams

can be used when you need:

- to defer processing until you need the data
- to deal with large collections, big files or remote resources<br>
  (when you don't need all the data at once or just cannot get all
  the data physically - like in case of remote resources)

to get lazy behaviour when dealing with collections just replace `Enum` module
with `Stream` one (since all stream implement `Enumerable` protocol) and call,
say, `Enum.to_list/0` or `Enum.take/1` in the end:

```sh
iex> [1, 2, 3, 4, 4] |> Enum.map(&(&1 * &1)) |> Enum.with_index
[{1, 0}, {4, 1}, {9, 2}, {16, 3}, {16, 4}]
iex> stream = [1, 2, 3, 4, 4] |> Stream.map(&(&1 * &1)) |> Stream.with_index
#Stream<[enum: [1, 2, 3, 4, 4],
 funs: [#Function<47.36862645/1 in Stream.map/2>,
  #Function<64.36862645/1 in Stream.with_index/2>]]>
iex> stream |> Enum.to_list
[{1, 0}, {4, 1}, {9, 2}, {16, 3}, {16, 4}]
```

benefit of using stream is that we don't store any intermediate results -
we just pass successive elements from one function to the next in the chain.

apart from `Stream` more and more modules now also support streams:

```elixir
IO.puts File.open!("test.txt") |> IO.stream(:line) |> Enum.to_list
# is equivalent to:
IO.puts File.stream!("test.txt") |> Enum.to_list
```

`Stream` functions to build your own streams:

- `Stream.cycle`

  cycles through collection elements infinitely:

  ```elixir
  Stream.cycle(~w{green white})
  |> Stream.zip(1..5)
  |> Enum.map(fn {class, index} ->
    ~s{<tr class="#{class}"><td>#{index}</td></tr>\n}
  end)
  |> IO.puts
  ```

- `Stream.repeatedly`

  calls supplied function each time new value is requested:

  ```sh
  iex> Stream.repeatedly(&:random.uniform/0) |> Enum.take(3)
  [0.4435846174457203, 0.7230402056221108, 0.94581636451987]
  ```

- `Stream.iterate`

  generates values infinitely using initial value and iteration function:

  ```sh
  iex> Stream.iterate(0, &(&1 + 1)) |> Enum.take(5)
  [0, 1, 2, 3, 4]
  ```

  useful to generate all kinds of progressions (arithmetic, geometric, etc.).

- `Stream.unfold`

  general form of iteration function:

  ```elixir
  fn state -> {stream_value, new_state} end
  ```

  almost same as above but:

  - iteration function has to be called to get the 1st stream value<br>
    (in `Stream.iterate` initial value is the 1st stream value)
  - stream value doesn't have to be equal to new state<br>
    (in `Stream.iterate` new state is current stream value)

  example:

  ```sh
  iex> Stream.unfold({0, 1}, fn {n1, n2} -> {n1, {n2, n1 + n2}} end) |> Enum.take(8)
  [0, 1, 1, 2, 3, 5, 8, 13]
  ```

- `Stream.resource`

  this function builds upon `Stream.unfold` and is intended to work with
  resources.

  it has 3 arguments:

  - function to get resource (instead of initial value in `Stream.unfold`)
  - iteration function
  - function to deallocate resource

  ```elixir
  Stream.resource(fn -> File.open!("test.txt") end,
                  fn file ->
                    case IO.read(file, :line) do
                      data when is_binary(data) -> {[data], file}
                      _ -> {:halt, file}
                    end
                  end,
                  fn file -> File.close(file) end)
  ```

## comprehensions

they are used to map and/or filter collections.

general form:

```
result = for generator{, generator}{, filter}[, into: value], do: expression
generator = pattern <- enumerable_thing
```

example (it works like nested for loops do):

```sh
iex> first8 = [1, 2, 3, 4, 5, 6, 7, 8]
iex> for x <- first8, y <- first8, x >= y, rem(x * y, 10) == 0, do: {x, y}
[{5, 2}, {5, 4}, {6, 5}, {8, 5}]
```

using comprehensions with binaries:

```sh
iex> for <<char <- "hello">>, do: char
'hello'
```

`into` parameter allows to save the result of comprehensions into specified
collection (it must implement `Collectable` protocol):

```sh
iex> for x <- ~w{cat dog}, into: %{}, do: {x, String.upcase(x)}
%{"cat" => "CAT", "dog" => "DOG"}
```

## binaries

binary is a bitstring (sequence of bits) where the number of bits is divisible
by 8 (that is each term occupies 1 byte). it has the form:

```
<<term[::modifier], ...>>
```

excess bits in binary are stripped:

```sh
iex> <<255>>
<<255>>
iex> <<256>>
<<0>>
iex> <<257>>
<<1>>
```

if you change the size of term with modifier it's no longer a binary:

```elixir
<<256>> # binary (size is 8 bits) => <<0>>
<<256::size(9)>> # bitstring (size is 9 bits) => <<128, 0::size(1)>>
```

```elixir
# integers
<<0, 1, 255>>
# integers with size modifier
<<1::size(3), 2::size(2)>> # 001 10 => <<6::size(5)>>
# floats
# https://www.doc.ic.ac.uk/~eedwards/compsys/float/
<<2.5::float>> # <<64, 4, 0, 0, 0, 0, 0, 0>>
```

### strings

string (aka dqs - double-quoted string) is UTF-8 encoded binary.

from <https://hexdocs.pm/elixir/String.html>:

- codepoint is a single Unicode character
  (may be represented by 1+ bytes)
- grapheme can consist of multiple codepoints
  (that may be perceived as a single character by readers)

```sh
iex> string = "\u0065\u0301"
iex> byte_size(string)
3
iex> String.length(string)
1
iex> String.codepoints(string)
["e", "́"]
iex> String.graphemes(string)
["é"]
```

#### strings vs. character lists

- "hello" - string
- 'hello' - character list

both strings and character lists:

- can hold characters in UTF-8 encoding
- may contain escape sequences
- allow interpolation
- allow to escape special characters with backslash
- support heredocs

### heredocs

heredoc is string contents delimited with triple single (`'''`) or
double (`"""`) quotes on separate lines:

```elixir
IO.puts """
  hello
  world!
  """
```

NOTE: trailing delimiter must be indented to the same level as the contents.

heredocs are used extensively to add documentation for functions and modules.

### sigils

sigil is a symbol with magical powers - it starts with a tilde, followed by
a letter (sigil type), delimited content and optionally some modifiers.

possible delimiters: `<>, {}, [], (), ||, //, "", ''`

sigil types:

| sigil type  | description
|-------------|------------------------------------------------
| ~C          | character list without escaping and interpolation
| ~c          | character list with escaping and interpolation
| ~D          | Date in format yyyy-mm-dd
| ~N          | naive DateTime in format yyyy-mm-dd dd:mm:ss[.ddd]
| ~R          | regexp without escaping and interpolation
| ~r          | regexp with escaping and interpolation
| ~S          | string without escaping and interpolation
| ~s          | string with escaping and interpolation
| ~T          | Time in format hh:mm:ss[.dddd]
| ~W          | list of words without escaping and interpolation
| ~w          | list of words with escaping and interpolation

modifiers for ~W and ~w sigils:

| modifier  | sigils returns
|-----------|------------------------------------------------
| a         | list of atoms
| c         | list of character lists
| s         | list of strings

modifiers for ~R and ~r sigils:

| modifier  | meaning
|-----------|------------------------------------------------
| f         | force pattern to start to match on the 1st line of miltiline string
| g         | support named groups
| i         | make matches case insensitive
| m         | ^ and $ match start and end of lines of multiline string
| s         | allow . to match newline characters
| U         | make * and + modifiers ungreedy (same as *? and +?)
| u         | enable unicode-specific patterns like \p
| x         | extended mode (ignore whitespaces and comments)
