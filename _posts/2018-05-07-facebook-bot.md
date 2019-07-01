---
layout: post
title: Facebook - Bot
date: 2018-05-07 09:46:40 +0300
access: public
comments: true
categories: [facebook]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>FD</dt>
  <dd>Facebook for Developers</dd>

  <dt>PAT</dt>
  <dd>page access token</dd>

  <dt>ATD</dt>
  <dd>Access Token Debugger</dd>
</dl>

<hr>

1. <https://developers.facebook.com/docs/messenger-platform>
2. <https://www.youtube.com/playlist?list=PLGhhbubwuDyTJJffHoLtIiQZYlv3XbUl->

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

configure webhook
-----------------

1. <https://www.youtube.com/watch?v=ZlysN-027Q4&index=2&list=PLGhhbubwuDyTJJffHoLtIiQZYlv3XbUl->
2. <https://developers.facebook.com/docs/messenger-platform/introduction/integration-components>
3. <https://developers.facebook.com/docs/messenger-platform/webhook>

> <https://developers.facebook.com/docs/messenger-platform/introduction/integration-components>
>
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

| FD: `PRODUCTS` (section in left menu) → `Webhooks`
| `Edit Subscription` (button)

### subscribe webhook to page events

| FD: `PRODUCTS` (section in left menu) → `Messenger` → `Settings`
| `Webhooks` (section) → `Select a Page` (combobox) → `Subscribe` (button)

generate page access token
--------------------------

PAT is required to send messages on behalf of selected page.

see [Facebook - API]({% post_url 2018-05-23-facebook-api %}) on how to
generate PATs in either FD or GAE.

configure messenger
-------------------

### get started button

1. <https://developers.facebook.com/docs/messenger-platform/reference/messenger-profile-api/get-started-button>
2. <https://www.techiediaries.com/messenger-bot-get-started-button/>

make sure you Messenger app is subscribed to `messaging_postbacks` event:

| FD: `PRODUCTS` (section in left menu) → `Messenger` → `Settings`
| `Webhooks` (section) → `Edit events` (button)

set up account linking
----------------------

1. <https://developers.facebook.com/docs/messenger-platform/identity/account-linking>
2. <http://blog.99array.com/2017/05/28/facebook-account-linking/>

> <https://medium.com/@philippholly/bbe632c578ca>
>
> AFAIK yes, you can open a webview with your own hosted website where you grab
> the messenger user id, and tell the user to click on “login with facebook”.
> Then you get the APP ID and can save a relation in your database with the
> messenger user id.

testing
-------

you must be app admin (not page admin) to receive webhooks events when
user sends a message to a page associated with your Messenger app (bot):

| FD: `PRODUCTS` (section in left menu) → `Messenger` → `Settings`
| `Token Generation` (section)

> Page token is required to start using the APIs. This page token will have
> all messenger permissions even if your app is not approved to use them yet,
> though in this case you will be able to message only app admins.

you'll be able to message users having ANY app role (admin, developer, tester)
- not necessarily admin role.

notes
-----

### about ASID vs. PSID

> <https://developers.facebook.com/docs/messenger-platform/identity/id-matching>
>
> When a person uses Facebook Login on a website or a mobile app, an ID
> is created for the specific Facebook app, which is called app-scoped ID.
> When a person interacts with a business via Messenger, an ID is created
> for the specific Page associated with the bot in Messenger, which is
> called Page-scoped ID.

NOTE: PSID is the same for all applications subscribed to this page messages.

troubleshooting
---------------

### Account Linking Failed

I see the error in browser when trying to complete account linking flow.

**solution**

1. <https://developers.facebook.com/docs/messenger-platform/reference/webhook-events/messaging_account_linking>

I haven't subscribed my webhook to `messaging_account_linking` page event =>
add `messaging_account_linking` page subscription field in Messenger settings:

| FD: `PRODUCTS` (section in left menu) → `Messenger` → `Settings`
| `Webhooks` (section) → `Edit events` (button)

> Edit Page Subscription Fields

- [x] `messaging_account_linking`

### Error validating access token: The user has not authorized application

I got this error when I tried to send login button to user on behalf of
Messenger app (chat bot which sends and receives messages using Facebook
Page) using PAT => this access token appears to be invalid.

**solution**

1. <https://developers.facebook.com/docs/facebook-login/access-tokens/debugging-and-error-handling#deauthorizedapp>

the point is that I have de-authorized Messenger app (by removing business
integration in Facebook account settings) making token generated for this
page invalid.

ATD shows error for this PAT now:

```
Error validating access token: The user has not authorized application <app_id>
```

solution is to authorize application once again by generating new PAT:

| FD: `PRODUCTS` (section in left menu) → `Messenger` → `Settings`
| `Token Generation` (section) → `Page` (combobox)

select required page and copy generated PAT into Messenger app.
