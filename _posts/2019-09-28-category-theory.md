---
layout: post
title: Category Theory
date: 2019-09-28 23:08:51 +0300
access: private
comments: true
categories: []
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

RATIONALE

That is why would you want to study category theory and why and how it can be
useful to you - future programmers and engineers. Let us be honest - very few of
you will dedicate your life to science. And frankly speaking Tula State
University prepares engineers first of all - not scientists. This is not bad -
it's just the way it is.

I want to start with a quote - it's just a comment to YouTube video (write it on
the blackboard):

> <https://www.youtube.com/watch?v=I8LbkfSSR58&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_>
>
> Me when I was a student: Who cares about theory, I want to learn Java and get
> a job
>
> Me today: God I wish I wasn't so stupid back then

IDK if you realize it now or not - but you need to learn some theory. And not
somewhere in the future but right now. Because you will have no time in the
future to do it thoroughly. You'll get a job, start a family, start jogging,
etc. Your spare time will be decreasing every year. So now is the best time to
learn theory and category theory in particular.

Why category theory? Because you will encounter lots of terms from category
theory in your daily life as software developers. Finally if you opt to learn
and use functional programming languages you will have to deal with a lot of
concepts from category theory. Of course you don't have to know the math to use
some programming primitives but having this knowledge will give you advantage
over you collegues. And you will thank yourself that you spent some time
learning the basics here in the university.

My course will be based on the book and video lectures by Bartosz Milewski. As
you might guess he is some Polish guy judging by his surname. He got his PhD in
Theoretical Physics in Poland. He worked in Microsoft, then founded his own
company, wrote a book "C++ in Action". His interest in C++ template
metaprogramming and concurrency naturally led him to Haskell and then to
category theory. His series of blog posts was converted to the book "Category
Theory for Programmers" which became quite popular. This book is topic of our
course.

IDK whether you are familiar with reading technical literature in English but
IMO it's quite straightforward compared to reading fiction: vocabulary is quite
limited and you will learn most of it after reading, say, the first 50 pages.
This book is no exception. I encourage you to read it by yourself and ask
questions if you have a hard time understanding anything from the book.

The last but the least as the proverb goes - about my motivation. I must have
the one too. Of course the money I get by giving this course of lectures is no
motivation - I guess you understand it. First of all I'm interested and highly
motivated to learn the basics of category theory by myself - even though I
encounter its concepts quite often I have never found time to study the theory
behind them. This is the very lack of time I was talking about. You know how it
happens - you always have a long list of books (TODO list) of books to read. In
you next life ) Because in current life you are too busy making other more
important things. So I want to solidify my knowledge of this topic by myself
because the best way to learn anything is to try to explain it to someone else -
to you in this case :)

Also I'd like to clarify that I am neither professional teacher or professional
scientist - I am a professional software developer. That is I earn my living by
writing programs in different programming languages. That is why I ask you to be
lenient whenever possible.

## Book

### Preface

> You might be allergic to calculus or algebra, but it doesn't mean you won't
> enjoy category theory. Category theory is the kind of math that is
> particularly well suited for the minds of programmers. That's because category
> theory — rather than dealing with particulars — deals with structure.

> Composition is at the very root of category theory — it's part of the
> definition of the category itself. And I will argue strongly that composition
> is the essence of programming. We've been composing things forever, long
> before some great engineer came up with the idea of a subroutine. Some time
> ago the principles of structural programming revolutionized programming
> because they made blocks of code composable. Then came object-oriented
> programming, which is all about composing objects. Functional programming is
> not only about composing functions and algebraic data structures — it makes
> concurrency composable.

> Since this is category theory for programmers I will illustrate all major
> concepts using computer code. You are probably aware that functional languages
> are closer to math than the more popular imperative languages. They also offer
> more abstracting power.

There will be a lot of examples in Haskell so we will take time to learn its
basics. But the book itself also contains a plenty of examples in C++ though
they are more verbose and lack expressive power of Haskell.

In this book Haskell is used for sketching and documenting ideas. You don't have
to learn any advanced concepts - basic syntax only. It's gonna be enough for the
purpose of studying this book.

> Preface
>
> If you’re an experienced programmer, you might be asking yourself: I’ve been
> coding for so long without worrying about category theory or functional
> methods, so what’s changed? Surely you can’t help but notice that there’s been
> a steady stream of new functional features invading imperative languages. Even
> Java, the bastion of object-oriented programming, let the lambdas in. C++ has
> recently been evolving at a frantic pace — a new standard every few years —
> trying to catch up with the changing world.

JavaScript too has many functional libraries - Ramda is one of the most popular
ones.

I can give one example from my programming practice - it's monad. You will
encounter this buzzword in a lot programming languages - some of them support
them natively (mostly functional programming languages like Haskell) and it's
available as a separate package or library in other languages (say, `dry-monads`
in Ruby or `witchcraft` in Elixir). A lot of people speak about monads but it
looks like no one knows for sure what they are exactly. So we're going to fix it
or at least to give enough tools and information to find it out by yourself :)

One more thing about why to use functional languages:

> One of the forces that are driving the big change is the multicore revolution.
> The prevailing programming paradigm, object oriented programming, doesn’t buy
> you anything in the realm of concurrency and parallelism, and instead
> encourages dangerous and buggy design.

What concerts concurrency, Elixir (even though it's not a pure functional
language like Haskell) offers a lot of concurrent primitives and has OTP -
standard library inherited from Erlang. OTP makes it very easy to write
concurrent programs and what is important - to make it right. You don't have to
worry mutexs and atomic operations - it's all built into the language.

Say, Java also offers a lot of tools to write concurrent programs but still it
hurts and most programmers avoid using these tools. At least this was the case
when I used Java at my work more than 6 years ago )

> But even in the absence of concurrency... side effects are getting out of
> hand. It’s not that side effects are inherently bad — it’s the fact that they
> are hidden from view that makes them impossible to manage at larger scales.
> Side effects don’t scale, and imperative programming is all about side
> effects.

### 1 Category: The Essence of Composition

> A category is an embarrassingly simple concept. A CATEGORY consists of objects
> and arrows that go between them. That’s why categories are so easy to
> represent pictorially. An object can be drawn as a circle or a point, and an
> arrow... is an arrow. But the essence of a category is COMPOSITION. Arrows
> compose, so if you have an arrow from object 𝐴 to object 𝐵, and another arrow
> from object 𝐵 to object 𝐶, then there must be an arrow — their composition —
> that goes from 𝐴 to 𝐶.

#### 1.1 Arrows as Functions

> Think of arrows, which are also called MORPHISMS, as functions. You have a
> function 𝑓 that takes an argument of type 𝐴 and returns a 𝐵. You have another
> function 𝑔 that takes a 𝐵 and returns a 𝐶. You can compose them by passing the
> result of 𝑓 to 𝑔. You have just defined a new function that takes an 𝐴 and
> returns a 𝐶.
>
> In math, such composition is denoted by a small circle between functions: 𝑔 ∘
> 𝑓. Notice THE RIGHT TO LEFT ORDER of composition.

Tell students about pipe operator in Unix and Elixir which both go from left to
right.

> But in math and in Haskell functions compose right to left. It helps if you
> read 𝑔 ∘ 𝑓 as “g after f”.

I mentioned Ramda Javascript library before - it has function `compose`:

> <https://ramdajs.com/docs/#compose>
>
> Performs right-to-left function composition. The rightmost function may have
> any arity; the remaining functions must be unary.

```javascript
R.compose(
  Math.abs,
  R.add(5),
  R.multiple(3),
)(-2); // => 30
```

It is even much simpler in Haskel:

> Here’s the declaration of a function from A to B:
>
> `f :: A -> B`
>
> `g :: B -> C`
>
> Their composition is:
>
> `g . f`

> So here’s the first Haskell lesson: Double colon means “has the type of...”. A
> function type is created by inserting an arrow between two types. You compose
> two functions by inserting a period between them (or a Unicode circle).

#### 1.2 Properties of Composition

TODO

## Video lectures

### 1.1: Motivation and Philosophy

### 1.2: What is a category?

> https://ncatlab.org/nlab/show/Set
>
> Set is the (or a) category with sets as objects and functions between sets as
> morphisms.
