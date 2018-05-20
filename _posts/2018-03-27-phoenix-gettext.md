---
layout: post
title: Phoenix - Gettext
date: 2018-03-27 14:30:24 +0300
access: public
comments: true
categories: [phoenix, gettext]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <http://blog.plataformatec.com.br/2016/03/using-gettext-to-internationalize-a-phoenix-application/>

configuration
-------------

1. <https://hexdocs.pm/gettext/Gettext.html#module-default-locale>

```elixir
# config/config.exs

config :billing, BillingWeb.Gettext,
  default_locale: "ru"
```

dynamic translations
--------------------

### create default POT file

```po
## priv/gettext/default.pot (create if missing)

msgid "system_error"
msgstr ""
```

_priv/gettext/default.pot_ might be missing if you add translations for the
first time (it's generated automatically when running `mix gettext.extract`
task - but only if there exists at least one call to `gettext` in your code).

NOTE: only dynamic translations are added manually to POT file:

> Add new translations manually only if they're dynamic
> translations that can't be statically extracted.

### generate PO files for all present locales

PO files are generated for all locales which are present in _priv/gettext/_
(only `en` locale is present by default):

```sh
$ mix gettext.merge priv/gettext
```

### add translations for all present locales

```diff
  ## priv/gettext/en/LC_MESSAGES/default.po

  msgid ""
  msgstr ""
  "Language: en\n"
  "Plural-Forms: nplurals=2\n"

  msgid "system_error"
+ msgstr "system error"
```

### add new locale

```sh
$ mix gettext.merge priv/gettext --locale ru
```

### add translation for new locale

```diff
  ## priv/gettext/ru/LC_MESSAGES/default.po

  msgid ""
  msgstr ""
  "Language: ru\n"
  "Plural-Forms: nplurals=3\n"

  msgid "system_error"
+ msgstr "системная ошибка"
```

### translate text dynamically in code

say, in a view module helper:

```elixir
# api_error_enum = "system_error"
def human_api_error(api_error_enum) do
  Gettext.gettext(BillingWeb.Gettext, api_error_enum)
end
```
