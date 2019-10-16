---
layout: post
title: Category Theory
date: 2019-09-28 23:08:51 +0300
access: private
comments: true
categories: [lecture]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## Introduction

### Rationale

Also I'd like to clarify that I am neither professional teacher nor professional
scientist - I am a professional software developer. That is I earn my living by
writing programs in different programming languages. That is why that is why you
might know many things better than me because I studied them many years ago
while you are studying right now. What I’m going to talk about with you may be
not mathematically strict so pardon for sloppiness. Probably the one thing I
know better than most of you is English. Though it’s also not perfect either of
course )

Our primary goal is for you to get accustomed to listening to and understanding
English speech - technical text in particular. This is first. Second, it would
be great if you memorize anything useful from this course. But exactly in this
order, not vice versa: technical English first, knowledge of the subject second.
That is why feel free to ask me about any words or expressions in my speech that
you don't know or about anything about English :) This is why we're here.

IDK whether you are familiar with reading technical literature in English but
IMO it's quite straightforward compared to reading fiction: vocabulary is quite
limited and you will learn most of it after reading, say, the first 50 pages.
Our course will be basde on one book and this book is no exception. I encourage
you to read it by yourself and ask questions if you have a hard time
understanding anything from the book. I will say a few words about the book
later.

The last but the least as the proverb goes - about my motivation. I must have
the one too. Of course the money I get by giving this course of lectures is no
motivation - I guess you understand it. First of all I'm interested and highly
motivated to learn the basics of category theory by myself - even though I
encounter its concepts quite often I have never found time to study the theory
behind them. You know how it happens - you always have a long list of books
(TODO list) of books to read. In you next life ) Because in current life you are
too busy making other more important things. So I want to solidify my knowledge
of this topic by myself because the best way to learn anything is to try to
explain it to someone else - to you in this case :)

### Structure of the course

This course will be based on the book and video lectures by Bartosz Milewski. As
you might guess he is some Polish guy judging by his surname. He got his PhD in
Theoretical Physics in Poland. He worked in Microsoft, then founded his own
company, wrote a book "C++ in Action". His interest in C++ template
metaprogramming and concurrency naturally led him to Haskell and then to
category theory. His series of blog posts was converted to the book "Category
Theory for Programmers" which became quite popular. This book is a topic of our
course. This book is accompanied by video lectures on YouTube which we'll also
be using.

Our goal is to study the 1st part of the book - it's a challenging task but we
have approximately 10 classes in this semester so we have all chances to do it.

Finally this book has a lot of examples in Haskell. It's HIGHLY likely that you
don't Haskell - just like me. But still it's quite popular languages among
certain groups of people. So we're going to take time to learn its basics. Some
concepts will be introduced as we study the book. Maybe we'll dedicate a whole
lesson to learning its syntax and basic constructs.

## Preface

There is a huge gap between science and engineering and consequently there is a
fear to learn math but fear not: it's not that scary in case of CT.

### Learn theory

You need to learn some theory. And not somewhere in the future but right now.
Because you will have no time in the future to do it thoroughly - unless it's
part of your job of course. You'll start a family, start jogging, spend all your
time at work, etc. Your spare time will be decreasing every year. So now is the
best time to learn some theory and CT in particular - why not?

Frankly speaking very of you (if any) will dedicate your life to science: TSU
prepares engineers first of all, not scientists. It's neither good nor bad -
it's just the way it is. So most likely you'll become developers and engineers
in the future. But even then knowing the basics of CT wouldn't do you any harm -
we will elaborate on this topic later when talking about FP (reason #4 to study
FP).

Let me conclude with a comment from YouTube video:

> <https://www.youtube.com/watch?v=I8LbkfSSR58&lc=UgxaeS0T_UIyupvXh9V4AaABAg>
>
> Me when I was a student: Who cares about theory, I want to learn Java and get
> a job.
>
> Me today: God I wish I wasn't that stupid back then.

### Reasons to learn CT

- it's a trove of extremely useful programming ideas
- it's particularly well-suited for the minds of programmers because CT deals
  with structure
- both CT and programming are about composing things
- we will use informal reasoning to make CT palatable to programmers

### Haskell

Haskell has more abstractive power than C++ => there will be a lot of examples
in Haskell. So we might take time to learn its basics. The book itself contains
plentry of examples in C++ though they are more verbose and lack expressive
power of Haskell. Haskell allows to express ideas at a higher level of
abstraction than imperative languages can do - this is what helps illustrate
ideas of CT which is the most abstract branch of math.

In this book Haskell is used for skething and documenting ideas. You don't have
to learn any advanced concepts - basic syntax only. It's gonna be enough for the
purpose of studying this book.

### Highest possible abstraction

CT is the highest possible level of abstraction, it's a higher-level abstraction
than Haskell, than any FP language. Ideas percolate from CT to Haskell and from
Haskell to other languages. These ideas cannot be programmed in CT because it's
not a programming language per se but they can be implemented in one of those
languages. When you learn CT, everything starts looking similar and you find out
common patterns in different branches of math and in programming - this is like
you're standing on top of Mount Everest.

### Why care about CT?

CT is closely related to FP and FP has the following benefits:

- FP becomes popular again, imperative languages (Java, JS) introduce functional
  features

  One example of such a feature is a monad. Some programmings languages support
  monads natively (say, Haskell) while others provide separate packages or
  libraries. Say, `dry-monads` gem in Ruby or `witchcraft` package in Elixir.
  Many people speak about monads but no one knows for sure what they are
  exactly. Basically because no one cares as long as monads fulfil their role.

  So whether you want or not, you opt to learn and use FP languages, you'll have
  to deal with concepts from CT. No doubt having this knowledge under your belt
  will give you a competitive advantage.

- multicore revolution

  Nothing is shared in FP and all data structures are immutable - this allows us
  to compose and decompose things without worrying about data races.

  Elixir (even though it's not a pure FP language like Haskell) offers plenty of
  concurrent primitives. Thanks to OTP it's easy to write concurrent programs
  and (what is most important) to make it right. You don't have to worry about
  mutexes and atomic operations - it's all built into the language.

  Java also offers tools to write concurrent programs but still it hurts => most
  developers avoid using these tools. At least this was the case when I used
  Java at my work more than 6 years ago :)

- side effects are limited and controlled

  Pure functions are preferred. Side effects are not hidden from view like in
  OOP.

## 1 Category: The Essence of Composition

Category consists of objects and arrows that go between them. Arrows are called
**morphisms**. It's a general term, kind of transformation but it's easy to
think of arrows as functions: functions are just one type of morphisms.

The essence of category is composition. Arrows compose: if there are arrows A →
B and B → C then arrow A → C is their composition which is denoted by a small
circle between functions (say, g ○ f).

Notice the right to left order of composition - it's just a tradition in both
math and in Haskell (so that the latter reads like math). We'll talk about how
it's different from composition associativity later.

I mentioned Ramda JS library before - it has function `compose`:

> Performs right-to-left function composition. The rightmost function may have
> any arity; the remaining functions must be unary.

```javascript
R.compose(
  Math.abs,
  R.add(5),
  R.multiple(3),
)(-2);
// => 30
```

### Properties of Composition

- Composition is associative

  > <https://en.wikipedia.org/wiki/Operator_associativity>
  >
  > In programming languages, the associativity of an operator is a property
  > that determines how operators of the same precedence are grouped in the
  > absence of parentheses.

  => associativity rules make sense only when we have 2 or more operators!

  In our case composition operator is a white circle (○).

  Parentheses indicate that composition (or any other operation) is to be
  performed first for the parentehsized functions.

  Operators can be (also they, say, `operator has the right associativity`):

  - associative

    operations are grouped arbitrarily: composition, multiplication and
    addition. if operations are both left and right associative, they are
    associative.

  - left-associative

    operations are grouped from the left: subtraction and division.

  - right-associative

    operations are grouped from the right.

  - non-associative

    operations cannot be chained, often because the output type is incompatible
    with the input types.

  don't confuse right to left order of composition with associativity of
  composition (which means it's both left-associative and right-associative).

  Proof that composition is associative.

  Let W, X, Y and Z be sets, and suppose that we are given functions:

  ```
  h: W → X, g: X → Y, f: Y → Z
  ```

  here W is called domain and X - codomain.

  We show that `(f . g) . h = f . (g . h)` as follows.

  Let w ∈ W (w is an element of, is a member of, is in, belongs to W). Then

  ```
  (( f . g) . h)(w) = (f . g)(h(w)) = f(g(h(w)))
  ```

  and

  ```
  (f . (g . h))(w) = f((g . h)(w)) = f(g(h(w)))
  ```

  Since `(( f . g) . h)(w) = (f . (g . h))(w)` for all w ∈ W, we have

  ```
  ( f . g) . h = f . (g . h)
  ```

- For every object A there is an arrow which is a unit of composition

  This arrow loops from the object to itself. The unit arrow for object A is
  called **identity** on A.

  In case of function identity arrow is implemented as identity function.

  Identity function is not that useful in daily life but it's useful as an
  argument to, or a return from, a higher-order function.
