---
layout: post
title: "notes about chef"
date: 2016-01-06 00:42:00 +0300
access: public
categories: [chef]
---

core principles:

* *idempotence*

* *thick clients, thin server*
  <br>
  chef does as much work as possible on the node and as little as possible on the server

* *order matters*
  <br>
  within a recipe resources are applied in the order they are listed

**chef-client** is installed on each node managed by chef server and is used to bring the node into the expected state
