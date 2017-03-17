---
layout: post
title: GA and GAW refresh and access tokens
date: 2017-03-17 15:52:21 +0300
access: public
categories: [ga, gaw]
---

GA and GAW use both refresh and access tokens however there are differences in
how we use them in our apps.

- GA

  we don't use any oficial google gem and provide both tokens manually when
  making API call to GA (access token is stored in Rails cache and manually
  updated when expired).

- GAW

  we use `google-adwords-api` gem to make API calls to GAW and only refresh token
  must be provided to configure API client - access token is updated by gem itself.
