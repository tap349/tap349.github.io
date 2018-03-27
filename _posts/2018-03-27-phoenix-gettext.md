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

_config/config.exs_:

```elixir
# gettext
config :billing, BillingWeb.Gettext,
  default_locale: "ru"
```

dynamic translations
--------------------

### create default POT file

create _priv/gettext/default.pot_ if it's missing:

```po
msgid "system_error"
msgstr ""
```

_priv/gettext/default.pot_ might be missing if you add translations for the
first time (it's generated automatically when running `mix gettext.extract`
task - but only if there exists at least one call to `gettext` in your code).

NOTE: only dynamic translations are added manually to POT file:

> Add new translations manually only if they're dynamic
> translations that can't be statically extracted.

### generate PO files

PO files are generated for all locales which are present in _priv/gettext/_:

```sh
$ mix gettext.merge priv/gettext
```

### translate text dynamically

say, in a view module helper:

```elixir
# api_error_enum = "system_error"
def human_api_error(api_error_enum) do
  Gettext.gettext(BillingWeb.Gettext, api_error_enum)
end
```

### add new language

```sh
$ mix gettext.merge priv/gettext --locale ru
```

### add translation for new language

_priv/gettext/ru/LC_MESSAGES/default.po_:

```diff
  msgid ""
  msgstr ""
  "Language: ru\n"
  "Plural-Forms: nplurals=3\n"

  msgid "system_error"
+ msgstr "системная ошибка"
```
