---
layout: post
title: Facebook - Bot
date: 2018-05-07 09:46:40 +0300
access: private
comments: true
categories: [facebook]
---

<!-- more -->

* TOC
{:toc}
<hr>

- FD - Facebook for Developers

change language
---------------

1. <https://developers.facebook.com/help/translations/>

- go to main page (<https://developers.facebook.com/>)
- scroll to footer and click required language there in `LANGUAGES` section

best practices
--------------

1. <https://developers.facebook.com/docs/messenger-platform/introduction/general-best-practices>

> Embrace Structure
>
> While recognizing free-form typed responses can be valuable, it can also be
> challenging to implement and tedious for people interacting with your bot.
> Make use of buttons, quick replies, and the persistent menu to structure user
> input. This can help streamline interactions and clearly communicate expectations.

> Map out Interactions
>
> Do not use standalone questions. This could imply free form interaction and
> encourage people to respond in ways you do not support. If you do pose questions,
> add buttons with specific answers to the message for people to choose from.

webhook
-------

1. <https://www.youtube.com/watch?v=ZlysN-027Q4&index=2&list=PLGhhbubwuDyTJJffHoLtIiQZYlv3XbUl->
2. <https://developers.facebook.com/docs/messenger-platform/introduction/integration-components>
3. <https://developers.facebook.com/docs/messenger-platform/webhook>

<https://developers.facebook.com/docs/messenger-platform/introduction/integration-components>:

> The Messenger Platform sends an event to your webhook whenever an action
> occurs in a conversation with your bot. Your webhook is a single HTTPS
> endpoint (usually /webhook) exposed by you that accepts POST requests.
> This is where your bot processes and responds to all incoming webhook events.
>
> The Messenger Platform supports a standard set of webhook events that you may
> subscribe your webhook to during the setup process. At a minimum, you should
> subscribe to the messages and messaging_postbacks webhook events to be able
> to implement basic Platform features in your bot.

### edit callback URL

1. <https://stackoverflow.com/a/37853597/3632318>

| FD: `PRODUCTS` (section in left menu) → `Webhooks` → `Edit Subscription` (button)

notes
-----

<https://developers.facebook.com/docs/messenger-platform/identity/id-matching>:

> When a person uses Facebook Login on a website or a mobile app, an ID
> is created for the specific Facebook app, which is called app-scoped ID.
> When a person interacts with a business via Messenger, an ID is created
> for the specific Page associated with the bot in Messenger, which is
> called Page-scoped ID.
