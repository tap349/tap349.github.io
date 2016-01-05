---
layout: post
title: notes about chef
date: 2016-01-06 00:42:00 +0300
access: public
categories: [chef]
---

summary from book 'Cooking infrastructure by Chef' by Alexey Vasiliev

<!-- more -->

##### core principles:

* *idempotence*
* *thick clients, thin server*
  <br>
  chef does as much work as possible on the node and as little as possible on the server
* *order matters*
  <br>
  within a recipe resources are applied in the order they are listed

**chef-client**

- installed on each node managed by chef server
- used to bring the node into the expected state

______
## Chef Solo
______

**chef-solo**

- open-source version of chef-client with limited functionality
- doesn't require access to server
- requires that a cookbook be on the same physical disk as the node
- doesn't support authentication and authorization

