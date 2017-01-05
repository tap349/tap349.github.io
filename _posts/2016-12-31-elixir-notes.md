---
layout: post
title: Elixir notes
date: 2016-12-31 13:05:42 +0300
access: public
categories: [elixir]
---

Elixir notes.

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

bytecode modules are not written to disk - only loaded in memory.

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
def for_location([head = [_, target_loc, _, _] | tail], target_loc) do
  [head | for_location(tail, target_loc)]
end
```

if `target_loc` argument matches the 2nd element of list head,
`head` variable matches the whole list head (is getting bound to it).

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
defp _sum([head | tail], total), do: _sum(tail, head + total)

# The Elixir Style Guide
defp do_sum([], total), do: total
defp do_sum([head | tail], total), do: do_sum(tail, head + total)
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

Elixir has 3 directives for modules, all of them are lexically scoped -
they are effective from the point they are encountered till the end of
enclosing scope.

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
  # alias multiple module at once
  alias Google.Adwords.{Importer, Parser}
  ```

- `require`

  make macro definitions available when code is compiled.

### module attributes

```elixir
defmodule MyList do
  @z_ascii_code 122

  def caeser([head | tail], n) when head <= @z_ascii_code + n do
    [head + n | caeser(tail, n)]
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

`Keyword` module is used to manipulate keyword lists,

keyword lists are usually used to store options passed to functions:

```elixir
def draw_text(text, options \\ []) do
  options = Keyword.merge(@defaults, options)
  ...
end
```


### char lists

char list is just a list of codepoints (integers) internally -
in IEx it's printed as a list of characters if all characters are printable.

add non printable character codepoint (say, `0`) to force print as codepoints:

```sh
iex> 'z' ++ [0]
[122, 0]
```

## maps

`Map` module is used to manipulate maps.

choosing between maps and keyword lists (both are dictionaries):

|                                 | map   | keyword list |
|:--------------------------------|:-----:|:------------:|
| use in pattern matching         | ✓     |              |
| allow duplicate keys            |       | ✓            |
| preserve order of elements      | ✓     |              |
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

  use `^` pin operator for keys to prohibit binding explicitly
  (= use current variable value and don't even try to rebind):

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

  def admin? when name != ""
    is_admin
  end
end
```

### nested accessors (from `Kernel` module)

<https://hexdocs.pm/elixir/Kernel.html>

they work with dictionaries (maps and keyword lists):

|                   | accessors                                     ||
|                   | macro                    | function            |
|-------------------|--------------------------|---------------------|
| get_in            | no                       | (dict, keys)        |
| put_in            | (path, value)            | (dict, keys, value) |
| update_in         | (path, fn)               | (dict, keys, fn)    |
| get_and_update_in | (path, fn)               | (dict, keys, fn)    |

macros are more concise but functions allow to determine the number
and set of keys at runtime =>
we can say that macros are static nested accessors while
functions are dynamic ones.

path in macros is extracted using accessed key in the dictionary:

```elixir
put_in(opts[:foo][:bar], :baz)
# or
put_in(opts.foo.bar, :baz)
# is equivalent to
put_in(opts, [:foo, :bar], :baz)
```
