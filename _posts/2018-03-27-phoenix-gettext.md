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

<dl>
  <dt>POT file</dt>
  <dd>
    POT files are just template files and the translations in them do not
    actually contain translated strings.
  </dd>
</dl>

<hr>

1. <http://blog.plataformatec.com.br/2016/03/using-gettext-to-internationalize-a-phoenix-application/>
2. <https://hexdocs.pm/gettext/Gettext.html>

configuration
-------------

1. <https://hexdocs.pm/gettext/Gettext.html#module-default-locale>

```elixir
# config/config.exs

config :billing, BillingWeb.Gettext,
  default_locale: "ru"
```

static vs. dynamic translations
-------------------------------

> <https://hexdocs.pm/gettext/Gettext.html#module-gettext-api>
>
> There are two ways to use gettext:
>
> - using macros from your own gettext module, like MyApp.Gettext
> - using functions from the Gettext module

=> use macros for static translations, use functions for dynamic ones.

static translations
-------------------

> <https://hexdocs.pm/gettext/Gettext.html#module-using-macros>
>
> Using macros is preferred as gettext is able to automatically sync the
> translations in your code with PO files. This, however, imposes a constraint:
> arguments passed to any of these macros have to be strings at compile time.

### add new locale

```sh
$ mix gettext.merge priv/gettext --locale km
```

### translate messages in templates

```slim
p = gettext("authentication success")
```

- `gettext/2` - for simple translations
- `dgettext/3` - for domain-based translations
- `ngettext/4` - for plural translations

NOTE: these templates MUST be rendered in controllers for `gettext.extract`
      task to work (otherwise translations won't be extracted).

### update POT file and PO files

- update POT file (extract all translations into POT files)

  ```sh
  $ mix gettext.extract
  ```

  this will update (or create) _priv/gettext/default.pot_ POT file
  for simple translations.

- update PO files (merge POT files into PO files)

  PO files are updated for all locales which are present in
  _priv/gettext/_ (only `en` locale is present by default):

  ```sh
  $ mix gettext.merge priv/gettext
  ```

both actions in one go:

```sh
$ mix gettext.extract --merge
```

### add translations for all locales

translations are added to PO files only (not POT files):

```diff
  ## priv/gettext/en/LC_MESSAGES/default.po

  msgid "authentication success"
- msgstr ""
+ msgstr """
+ foo
+ bar
+ """
```

```diff
  ## priv/gettext/km/LC_MESSAGES/default.po

  msgid "authentication success"
- msgstr ""
+ msgstr """
+ foo
+ bar
+ """
```

dynamic translations
--------------------

> <https://hexdocs.pm/gettext/Gettext.html#module-using-functions>
>
> If compile-time strings cannot be used, the solution is to use the functions
> in the Gettext module instead of the macros described above. These functions
> perfectly mirror the macro API, but they all expect a module name as the first
> argument.

### create POT file

POT file (_priv/gettext/default.pot_ in case of simple translations) might
be missing if you're adding dynamic translations for the first time:

```po
## priv/gettext/default.pot (create if missing)

msgid "system_error"
msgstr ""
```

POT file is created manually iff you're adding dynamic translations:

> <https://hexdocs.pm/gettext/Gettext.html#module-compile-time-features>
>
> using the gettext macros (as opposed to functions) allows gettext to
> operate on those translations at compile-time. This can be used to extract
> translations from the source code into POT files automatically (instead of
> having to manually add translations to POT files when they’re added to the
> source code).

and only dynamic translations are added to POT file manually:

> _priv/gettext/default.pot_
>
> Add new translations manually only if they're dynamic translations that
> can't be statically extracted.

### update PO files

```sh
$ mix gettext.merge priv/gettext
```

### add translations for all locales

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
