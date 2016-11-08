---
layout: post
title: create new GAW credentials
date: 2016-11-08 14:03:10 +0300
access: public
categories: [gaw]
---

steps to get new GAW credentials.

<!-- more -->

- <https://github.com/googleads/google-api-ads-ruby/wiki/API-access-using-own-credentials-%28installed-application-flow%29>

### create (or get) OAuth 2.0 credentials in Google API Console

- select project in top left menu (say, `adwords-api`)
- Credentials (left sidebar) -> Credentials -> Credentials (tab)

here you can find OAuth 2.0 client ID and client secret.

### get developer token in Google AdWords

we have only one developer token (obtained for manager account) for
all our projects (including umka).

- select manager account (top-level account that manages nested accounts)
- click gear icon (top right menu) -> Account settings
- AdWords API Center (left sidebar)

here you can find developer token.

### generate access and refresh tokens

- save
  [setup_oauth2.rb](https://github.com/googleads/google-api-ads-ruby/blob/master/adwords_api/examples/v201607/misc/setup_oauth2.rb)
  ruby script anywhere in filesystem
- create _adwords_api.yml_ file in home directory
  (this is where script above searches this file)
  with the following contents:

  ```yaml
  :authentication:
    :method: OAuth2
    :oauth2_client_id: <client_id>
    :oauth2_client_secret: <client_secret>
    :developer_token: <developer_token>
    :client_customer_id: <customer_id>
  ```

  `<customer_id>` (in the form `111-222-3333`) is either manager account ID
  or ID of any other account that can manage clients.

  from my experience generated access and refresh tokens can be used in
  conjunction with both specified account and its parent (manager) account.
  but tokens generated for manager account (`Центр клиентов I*`)
  couldn't be used for subordinate manager account (`re***`) when
  trying to find the latter using `ManagedCustomerService`
  (that is I used `re***` account ID as master_api_key and tokens generated
  for manager account, tried to find `re***` account using aforementioned
  service and got `AuthenticationError.CUSTOMER_NOT_FOUND` error) -
  all other API services worked as expected (magic!).

- run `ruby setup_oauth2.rb` and follow on-screen instructions

  as a result of executing this script new _adwords_api.yml_ will be generated
  with access and refresh tokens populated.
