---
layout: post
title: Phoenix - Contexts
date: 2018-12-03 02:02:22 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://michal.muskala.eu/2017/05/16/putting-contexts-in-context.html>

ramblings about module naming in Elixir and code organization
-------------------------------------------------------------

as far as I can see folks prefer nouns to verbs when naming modules.

in this regard module acts like a namespace that groups related actions together.
say, in Ruby I might have several operations `User::Create`, `User::Update`, etc.
with a single `call` method (so called function objects) - in Elixir most likely
I would have a single `User` module with several functions `create`, `update`,
etc. inside.

moreover `User` module might define a struct and have functions working with
this struct inside the same module - of course struct must be passed to each
such function (modules are stateless).

if some functions inside `User` module can be grouped further it's possible to
create another module (= namespace) that is nested inside `User` (but usually
stored in a separate file in _user/_ directory) - `User.Authorization` module
for example. and so on and on...

that is why IMO Elixir modules tend to be bigger than Ruby classes.

***UPDATE***

the same principles apply to contexts (new feature of Phoenix 1.3) - they
group common functionality together (contexts are top-level domains of your
applicaiton). contexts can be further divided into subdomains (say, I have
such subdomains named after schemas - they contain operations, queries, etc.
that refer to (= bounded by) that specific schema).

CQRS approach
-------------

1. <https://blog.lelonek.me/command-query-separation-in-elixir-ac742e60fc7d>

when using context most operations that constitute context public interface
and are used internally inside context are implemented inside context module
itself. complex operations can be extracted into their own modules (services,
operations) bounded by schemas as described above.

in CQRS approach it's recommended to separate read and write operations and
group them by related schema inside corresponding loaders and mutators. in
this case it's possible to expose only those operations via context public
interface (by importing these operations or by creating wrapper functions)
which are meant to be used outside current context (in other contexts or in
web part of Phoenix application). all other operations are considered to be
internal and should be used inside current context by calling functions from
required loaders and mutators directly.

implementing most operations in schema loaders and mutators and adding only
required ones to context public interface has the benefit of not polluting
the latter with operations which are not meant to be used outside current
context.

still it makes sense to extract complex operations (which might have private
helper functions) into their own modules (services, operations) within schema
namespace - just like when not using CQRS approach. these operations can use
different schema loaders and mutators and can be added to context interface.
for example, these operations can be placed inside `operations` namespace.

alternatively complex operations might be implemented as context functions
if they are just a sequence of calls to corresponding loaders and mutators.

breaking up a context into smaller pieces with defdelegate
----------------------------------------------------------

1. <http://www.petecorey.com/blog/2018/09/03/using-facades-to-simplify-elixir-modules>

context module might grow big - it's possible to split it into several modules
and then expose all their functions in a single "facade" context module using
`Kernel.defdelegate/2` macro.

using umbrella apps as bounded contexts
---------------------------------------

1. <https://blog.usejournal.com/implementing-bounded-contexts-in-elixir-25bb8a80bbca>

it's possible to create a separate app for each bounded context inside umbrella
project:

> - Create an umbrella project;
>
> - One app for each bounded context;
>
> - An app should never call internal code from another app;
>
> - Use a module on each app to act as a public API to exchange data between
>   apps, and also to call functions to mutate data;
>
> - An app should never return its internal data structures on public apis,
>   it should return only raw data like maps and lists;

these rules can be further applied to ordinary Phoenix contexts inside separate
apps within umbrella project or contexts inside non-umbrella project.

secondary contexts
------------------

1. <http://devonestes.herokuapp.com/a-proposal-for-context-rules>

currently all modules within context (primary context in terms of this article)
can be accessed directly, contexts interact with each other via context modules
which act as facades.

Devon Estes suggests isolating not only contexts but resources within contexts
as well by introducing secondary contexts which are named after resources in
plural form and contain both loader and mutator functions. I guess all resource
related operations should be proxied via these secondary contexts as well.

this way it becomes very easy, say, to move a resource to another primary
context but at the same time it increases complexity (IMO) because it makes
you create facades within facades (secondary contexts within primary ones)
instead of using resource related modules directly within primary contexts.

all in all this idea is worth paying attention to but for the time being I'll
stick to my current scheme with separate loaders and mutators for each resource
(basically loader + mutator = secondary context).
