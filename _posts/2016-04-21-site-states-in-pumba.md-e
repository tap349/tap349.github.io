---
layout: post
title: site states in pumba
date: 2016-04-21 12:02:20 +0300
access: private
categories: [pumba]
---

conventional scheme with state machine (SM) callbacks:

- call SM event to change site state (`start.start!`)
- create before callback for this transition (`before_transition`)

using site state services (event + transition logic):

- create state service whose name is similar to SM event name (`Site::Start`):
  - do some actions that are meant to be done in transition callback
  - call SM event to change site state (`start.start!`) -
    it must be the same event as implied by state service name (!)
  - optionally create task that might call another state service
    upon its completion.

if using site state services:

- never call SM event directly anywhere else in code
- never use transition callback in site model
