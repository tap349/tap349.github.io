---
layout: post
title: React Native - Subscriptions (iOS)
date: 2018-03-01 18:40:21 +0300
access: public
comments: true
categories: [react-native, ios]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>IC</dt>
  <dd>iTunes Connect</dd>

  <dt>IAP</dt>
  <dd>in-app purchase</dd>
</dl>

<hr>

1. [Workflow for configuring in-app purchases](https://help.apple.com/itunes-connect/developer/#/devb57be10e7)
2. [In-App Purchase Programming Guide](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Introduction.html)
3. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>
4. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>

implement subscriptions in application
--------------------------------------

1. <https://github.com/chirag04/react-native-in-app-utils>
2. <https://github.com/sibelius/iap-receipt-validator>
3. [Adding In-App Purchase to Your Applications](https://developer.apple.com/library/content/technotes/tn2259/_index.html)
4. [Validating Receipts With the App Store](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html)
5. [Receipt Fields](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html)

### obtain a shared secret

| IC: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `App-Specific Shared Secret` → `View Master Shared Secret` (link)

shared secret is used to validate receipts with `iap-receipt-validator`
npm package.

add subscription in IC
----------------------

### create IAP

| IC: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `In-App Purchases` (section) → `⨁` (link)

> Select the in-app purchase you want to create.

- [x] `Auto-Renewable Subscription`

> Create Auto-Renewable Subscription

- `Reference Name` (input): `No Ads Monthly`
- `Product ID` (input): `<bundle_id>.sub.noads.monthly`

  `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
  in _project.pbxproj_ - `com.iceperk.iceperkapp` for our application.

  product ID must be unique - even after deleting subscription
  its product ID cannot be reused for new subscriptions.

> Create Subscription Group

- [x] `Create New Subscription Group`
  - `Subscription Group Reference Name` (input): `No Ads`

    must be more general than subscription reference name.

### configure IAP

| IC: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `<subscription>` (link)

- [x] `Cleared for Sale`
- `Subscription Duration` (combobox): `1 Month`
- `Subscription Prices` (section) → `⨁` (link)
  - `Currency` (combobox): `RUB - Russian Ruble`
  - `Price` (combobox): `149 р.`
- `Localizations` (section) → `⨁` (link) → `Russian`
  - `Subscription Display Name` (input): `Скрытие рекламы на месяц`
  - `Description` (input): `Полное отключение рекламы в приложении` (not required)

<https://developer.apple.com/library/content/technotes/tn2259/_index.html#//apple_ref/doc/uid/DTS40009578-CH1-ITUNES_CONNECT>:

> Leave the state of your product as Missing Metadata.
>
> Upload a a screenshot of your in-app purchase product once you
> are done testing it and ready to upload it for review.

### configure subscription group

| IC: `My Apps` → `<my_app>` → `Features` (tab)
| `<subscription_group>` (link)

it's required to add localization for subscription group as well:

> Before you can submit your in-app purchase for review,
> you must add at least one localization to your subscription group.

- `Localizations` (section) → `⨁` (link) → `Russian`
  - `Subscription Group Display Name` (input): `Подписки`
  - `App Name Display Name`: [x] Use App Name

test subscription on real device
--------------------------------

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

it's required to run application on real device to make purchases:
the only action allowed from inside emulator is loading products.

test subscriptions:

- have shortened periods (say, 1 month → 5 minutes)
- renew 5 times only (cancelled automatically afterwards)
- cannot be managed (say, cancelled manually)

test subscriptions are not created separately - they are real subscriptions
but with special characteristics and behaviour for sandbox testers.

<https://forums.developer.apple.com/thread/14702>:

> the sandbox environment handles auto-renewing subscriptions in an automated
> fashion - that is subscription periods are shortened and renew only 5 times
> and not controlled through the subscription management screen.

### sandbox testing

#### add sandbox testers (sandbox tester accounts)

1. <https://support.magplus.com/hc/en-us/articles/203809008-iOS-How-to-Test-In-App-Purchases-in-Your-App>

| IC: `My Apps` → `Users and Roles` → `Sandbox Testers` (tab)
| `Testers` (section) → `⨁` (link)

tips and notes regarding sandbox tester:

- new sandbox tester shouldn't have an existing Apple ID

  I got `That email address is already associated with an existing Apple ID`
  error when I tried to add myself as a sandbox tester - Apple ID is created
  for each new sandox tester right after it's added here.

- sign out of Apple ID on iPhone before testing subscription

  1. <https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/ShowUI.html#//apple_ref/doc/uid/TP40008267-CH3-SW11>

  it's also important not to sign in as a sandbox tester in `Settings` but
  in application only (when prompted before making a purchase) - a sandbox
  tester account is not a fully functional one and tester's Apple ID cannot
  be used to access iTunes Store or App Store.

  occasionally `Apple ID Verification` alert dialog might appear asking
  to enter password for sandbox tester account in `Settings` - it's safe
  to ignore it and press `Not Now` button all the time.

##### about verification of sandbox tester email

generally speaking it's necessary to verify new sandbox tester account by
following the link sent to specified email:

> You must validate your e-mail, or any purchases you make will silently fail.

but I couldn't verify account of a new sandbox tester - after clicking
the link email verification page kept on loading forever with an activity
indicator in the middle of the screen and JS errors in Chrome Console:

```
bundle.js:46270 Refused to create a worker from 'blob:https://id.apple.com/...'
because it violates the following Content Security Policy directive...
```

IDK if it matters or not but I signed in to iCloud with this account
(in browser) and completed registration by adding 3 secret questions.

in the end I could make a purchase using not verified sandbox tester account.

***UPDATE (2018-03-19)***

I have successfully verified email of another sandbox tester -
most likely it was temporary technical problem on Apple side.

#### set either development or production environment in application

it's possible to purchase test subscriptions in both `development` and
`production` environments but read the caveats about validating receipt
from sandbox tester in `production` environment in the next section.

NOTE: there were lots of `Network request failed` errors when trying
      to purchase a test subscription - so keep on trying.

#### run application on real device

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})
2. <http://pinkstone.co.uk/deploying-your-app-from-xcode-to-a-device-with-release-build-configuration/>

sandbox tester can purchase a subscription in production release but
subscription status cannot be checked because sandbox receipt cannot
be validated in `production` environment (see `troubleshooting` section
below for details).

- run application in debug mode (default build configuration)

  | Xcode: `Product` (menu) → `Scheme` → `Edit Scheme...`
  | `Run` (left menu) → `Info` (tab)

  - `Build Configuration` (combobox): `Debug`
  - `Close` (button)

  | Xcode: `▶` (toolbar button)

- run application in release mode

  | Xcode: `Product` (menu) → `Scheme` → `Edit Scheme...`
  | `Run` (left menu) → `Info` (tab)

  - `Build Configuration` (combobox): `Release`
  - `Close` (button)

  | Xcode: `▶` (toolbar button)

#### troubleshooting

- `App installation failed. An unknown error has occurred.`

  <table>
    <tr>
      <th>where</th>
      <th>when</th>
    </tr>
    <tr>
      <td>Xcode modal window</td>
      <td>trying to run application on real device</td>
    </tr>
  </table>

  <https://stackoverflow.com/a/41138265/3632318>:

  - close Xcode
  - unplug device
  - uninstall application from device
  - open Xcode
  - clean build in Xcode
  - run application on device again

- `not_available`

  <table>
    <tr>
      <th>where</th>
      <th>when</th>
    </tr>
    <tr>
      <td>Log.error</td>
      <td>checking subscription status</td>
    </tr>
  </table>

  no receipt is found:

  - there is no signed in Apple ID

    wait for `Sign-In Required` modal window or sign in manually in
    iPhone's `Settings`.

  - current Apple ID has never made a purchase in this application

    process this error in application so that user is considered to
    be unsubscribed.

- `Sandbox receipt sent to Production environment`

  <table>
    <tr>
      <th>where</th>
      <th>when</th>
    </tr>
    <tr>
      <td>Log.error</td>
      <td>checking subscription status</td>
    </tr>
  </table>

  current environment is `production` and it's passed to `iapReceiptValidator`
  function of `iap-receipt-validator` package when trying to validate receipt
  of sandbox tester.

  so switch to `development` environment or send `development` environment
  to `iapReceiptValidator` function regardless of current environment.

- checking subscription status always fails in production release

  this is exactly the error above - you cannot validate receipt of
  sandbox tester in `production` environment.

  and this is why Apple reviewer couldn't test subscription:

  - he signs in as sandbox tester on his iPhone
  - he uses production release - so it's a `production` environment
  - by default subscription status is `true` (before any checks)
  - checking subscription status always fails for sandbox tester in
    production release
  - subscription status remains to be `true`
  - reviewer doesn't see subscribe button and decides that subscription
    functionality is not implemented
  - binary is rejected in IC :(

  solution? I guess production releases shouldn't be tested by sandbox testers at
  all since we can't tell sandbox tester from real user inside our application -
  it's not our application user but currently signed in Apple ID on iPhone itself.
  so we always have to assume `production` environment when validating receipt in
  production release.

  one thing we can do here is to change behaviour of our application and set
  default subscription status to `false` so that subscribe button is shown by
  default - all subsequent subscription status checks will fail so the button
  will always be shown until sandbox tester makes a purchase.

  after making a purchase subscription status should change to `true` though
  since this is done inside operation to purchase a subscription - not within
  operation to check subscription status which keeps on failing forever. and
  it's all okay until the user decides to restart application - subscription
  status is then reset to its default value `false` (which is incorrect) but
  this value won't change because of failing checks.

  <https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/AppReview.html>:

  > When validating receipts on your server, your server needs to be able
  > to handle a production-signed app getting its receipts from Apple’s test
  > environment. The recommended approach is for your production server
  > to always validate receipts against the production App Store first. If
  > validation fails with the error code “Sandbox receipt used in production”,
  > validate against the test environment instead.

### beta testing

| IC: `My Apps` → `<my_app>` → `TestFlight` (tab)
| `All Testers` (left menu)

beta testers (or just testers) are IC users who are invited to test
TestFlight builds - that is they must have been already added before.
just like sandbox testers they:

- deal with test subscriptions only (shortened periods, etc.)
- are not charged when making IAP

NOTE: beta testers can't buy subscription even if it's approved it IC.

#### install application from TestFlight

when beta tester is making a purchase, all Apple dialogs no longer
have the line `[Environment: Sandbox]` (like for sandbox testers).

publish release with subscription
---------------------------------

### fill review information

1. <https://stackoverflow.com/questions/4802810/iphone-in-app-purchase-screen-shot/4803012>
2. <https://developer.apple.com/app-store/review/guidelines/>
3. <https://help.apple.com/itunes-connect/developer/#/dev84b80958f>

| IC: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `<subscription>` (link) → `Review Information` (section)

- `Screenshot`: screenshot of your app with subscription popup
- `Review Notes`: details on how you work with subscription in your app

  notes might also include details about how you're going to monetize your
  application and some implementation details (the more the better I guess).

- `Save` (button)

after you fill review information, subscription status is automatically
changed from `Missing Metadata` to `Ready to Submit`.

### create new app version

| IC: `My Apps` → `<my_app>` → `App Store` (tab)
| `⨁ VERSION OR PLATFORM` (link in left menu) → `iOS` (popup menu)

- `Store Version Number`: new version number (say, `3.14`)
- `Create` button

new app version has `Prepare for Submission` status now.

### add IAP to new app version

`In-App Purchases` page:

> Your first in-app purchase must be submitted with a new app version.
> Select it from the app’s In-App Purchases section and click Submit.
>
> Once your binary has been uploaded and your first in-app purchase has
> been submitted for review, additional in-app purchases can be submitted
> using the table below.

1. <https://stackoverflow.com/a/42191936/3632318>

| IC: `My Apps` → `<my_app>` → `App Store` (tab)
| `3.14 Prepare for Submission` (left menu) → `In-App Purchases` (section) → `⨁` (link)

NOTE: `In-App Purchases` section is available only if you've created IAP before.

- select IAP (your subscription) in popup window

  > Select in-app purchases for us to review with this app version.
  > The in-app purchases shown below are the only ones in the Ready to Submit state.

- `Done` (button to select IAP and close popup window)
- `Save` (button to save changes to app version)

### release new app version

according to `iTunes Store` email received after releasing new app version:

```
It can take up to 24 hours before your app is available on the App Store.
This delay is dependent on any app availability issues
```

in fact it took about 30 minutes for new app version to appear in App Store.

### troubleshooting

- `Your app is using the Advertising Identifier (IDFA)`

  when trying to submit a binary:

  ```
  Your app is using the Advertising Identifier (IDFA).
  You must either provide details about the IDFA usage or
  remove it from the app and submit your binary again.
  ```

  <https://developers.google.com/admob/ios/download>:

  > The Mobile Ads SDK for iOS utilizes Apple's advertising identifier (IDFA).

  | IC: `My Apps` → `<my_app>` → `App Store` (tab)
  | `3.14 Prepare for Submission` (left menu) → `Submit for Review` (button)

  - `Does this app use the Advertising Identifier (IDFA)?`: [x] `Yes`
  - `This app uses the Advertising Identifier to (select all that apply):`
    - [x] `Serve advertisements within the app`
    - [ ] `Attribute this app installation to a previously served advertisement`
    - [ ] `Attribute an action taken within this app to a previously served advertisement`
  - [x] `I, <my_name>, confirm that this app...`
  - `Submit` (button)

- binary rejected (release didn't pass a review because of IAP)

  | IC: `My Apps` → `<my_app>` → `Activity` (tab)
  | `App Store Versions` (left menu) → `Resolution Center` (link)

  > **2.** 1 Performance: App Completeness
  > <br>
  > **3.** 1.2 Business: Payments - Subscriptions
  >
  > <h3>Guideline 2.1 - Performance - App Completeness</h3>
  >
  > We found that while you have submitted in-app purchase products for your app,
  > the in-app purchase functionality is not present in your binary.
  >
  > **Next Steps**
  >
  > If you would like to include in-app purchases in your app, you will need to
  > upload a new binary that incorporates the in-app purchase API to enable users
  > to make a purchase.
  >
  > Once you revise and resubmit your binary, you will also need to resubmit your
  > in-app purchases for review since they are in the Developer Action Required
  > state. For each in-app purchase product submitted, please be sure to edit the
  > detail information or cancel the request to change the detail information for
  > the in-app purchases using iTunes Connect.
  >
  > <h3>Guideline 3.1.2 - Business - Payments - Subscriptions</h3>
  >
  > We noticed that your app or its metadata did not fully meet the terms and
  > conditions for auto-renewing subscriptions, as specified in Schedule 2,
  > section 3.8(b) of the Paid Applications agreement.
  >
  > Your app's **binary** did not include:
  >
  > <ul>
  > <li> The following information about the auto-renewable nature of the subscription
  >   <ul>
  >   <li> Title of publication or service </li>
  >   <li> Length of subscription (time period and content or services provided
  >        during each subscription period) </li>
  >   <li> Price of subscription, and price per unit if appropriate </li>
  >   <li> Payment will be charged to iTunes Account at confirmation of purchase </li>
  >   <li> Subscription automatically renews unless auto-renew is turned off at
  >        least 24-hours before the end of the current period </li>
  >   <li> Account will be charged for renewal within 24-hours prior to the end of
  >        the current period, and identify the cost of the renewal </li>
  >   <li> Subscriptions may be managed by the user and auto-renewal may be turned
  >        off by going to the user's Account Settings after purchase </li>
  >   <li> Any unused portion of a free trial period, if offered, will be forfeited
  >        when the user purchases a subscription to that publication, where applicable </li>
  >   </ul>
  > </li>
  > <li> A link to the terms of use </li>
  > <li> A link to the privacy policy </li>
  > </ul>
  >
  > Note: Adding the above information to the StoreKit modal alert is not
  > sufficient; the information must also be displayed within the app itself,
  > and it must be displayed clearly and conspicuously during the purchase
  > flow without requiring additional action from the user, such as opening
  > a link.
  >
  > Your app's **metadata** did not include:
  >
  > <ul>
  > <li> The following information about the auto-renewable nature of the
  >      subscription (same subitems as for app's binary) </li>
  > <li> A link to the terms of use </li>
  > <li> A privacy policy link in the Privacy Policy URL field of iTunes Connect </li>
  > </ul>
  >
  > **Next Steps**
  >
  > To resolve this issue, please revise your app or its metadata to include
  > the missing information.
  >
  > If the above information is in your app, please reply to this message in
  > Resolution Center to provide details on where to locate it.

  1. <https://stackoverflow.com/a/43651411/3632318>
  2. <https://forums.developer.apple.com/thread/70917>
  3. <http://captaindanko.blogspot.ru/2017/06/addressing-app-review-rejections-for.html>

  what we made to resolve these issues:

  > 1 Performance: App Completeness

  I fixed an issue when sandbox tester receipt couldn't be validated in
  production environment - as a result subscription status checks didn't
  work and subscription remained active forever (it's active by default)
  effectively hiding subscription functionality (subscribe button).

  also I processed the situation when user had never made IAP before and thus
  had no receipt at all - `InAppUtils.receiptData` returns `not_available`
  error in this case and you should return subscription status `false` instead
  of raising error (the latter wouldn't change subscription status and user
  would remain subscribed).

  > 1.2 Business: Payments - Subscriptions

  we added this information to both subscription page in application and
  application description in IC:

  - subscription title
  - subscription length
  - subscription price
  - information about auto-renewable nature of subscription

  it's pretty standard and can be safely copied from description of
  any application in App Store that provides subscriptions.

  - link to privacy policy
  - link to terms of use

  in addition I edited review notes for our IAP to reflect new changes.

  after making all required changes it's necessary:

  - make sure IAP status is changed from `Developer Action Needed` to
    `Waiting for Review`

    1. <https://stackoverflow.com/a/7764496/3632318>

    | IC: `My Apps` → `<my_app>` → `Features` (tab)
    | `In-App Purchases` (left menu) → `<subscription>` (link)

    find the section that has a red circle mark - it's necessary to update
    information there. in my case it was `Localizations` section and I had
    a red circle next to `Russian` language.

    if you're not going to change anything and just want to re-submit IAP,
    edit rejected section (say, localized russian description in my case),
    save IAP, revert the changes and save again => red circle turns into a
    yellow one and IAP status becomes `Waiting for Review`.

  - upload new build, select it in current application version and submit
    the latter for review

    after that you can submit application version for review again.

- `In-App Purchases` section is gone in application version

  I found this out after submitting application for review (previous
  binary was rejected and IAP status was `Developer Action Needed`).

  after that we decided that IAP status was the reason why IAP section
  was missing in application version and that IAP wouldn't be included
  in a new release at all.

  thus we removed current application version from review and updated IAP
  status (see notes above) but IAP section still didn't appear so we had
  to submit application version for review just like before - without any
  mentions of IAP in application version.

  ***UPDATE (2018-03-24)***

  Apple has approved of our release eventually (it took 3 days for them to
  review our release):

  - iOS App status is changed to `Pending Developer Release`
    (we checked `Manually release this version` option)
  - IAP status is changed to `Approved`

view financial reports
----------------------

| IC: `App Analytics` → `Overview` (tab)
| `In-App Purchases` (widget) or `Sales` (widget)

| IC: `App Analytics` → `Metrics` (tab)
| `In-App Purchases` (left menu) or `Sales` (left menu)
