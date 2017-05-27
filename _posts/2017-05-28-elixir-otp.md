---
layout: post
title: Elixir - OTP
date: 2017-05-28 01:01:59 +0300
access: public
categories: [elixir, otp]
---

<!-- more -->

## GenServer vs. Agent

<https://groups.google.com/forum/#!topic/elixir-lang-talk/DCTXyGV791w>

> Agent is just a GenServer that only saves state.

> The benefit of an Agent over a GenServer is in the nomenclature.

Task is not a GenServer but you can use GenServer as a Task.
