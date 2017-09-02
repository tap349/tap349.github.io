---
layout: post
title: Phoenix - Troubleshooting
date: 2017-08-01 17:48:52 +0300
access: public
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>


## (FunctionClauseError) no function clause matching in Plug.Conn.put_resp_header/3

<https://github.com/ueberauth/guardian/issues/196>

response header value must be a string - convert `exp` variable to string:

```elixir
conn
|> put_resp_header("authorization", "Bearer #{jwt}")
|> put_resp_header("x-expires", "#{exp}")
# ...
```

## Slogan: Kernel pid terminated (application_controller)

_/home/billing/prod/billing/erl_crash.dump_ on production host:

{% raw %}
```
Slogan: Kernel pid terminated (application_controller) ({application_start_failure,billing,{{shutdown,{failed_to_start_child,'Elixir.BillingWeb.Endpoint',{#{'__exception__' => true,'__struct__' => 'Elixir.Run
```
{% endraw %}

**solution**

in most cases it means that `PORT` environment variable is not set - examine
`~/.profile` and `/etc/systemd/system/phoenix_billing.service` files.

## Node is not running! (Node not responding to pings.)

running application is not responding to ping/start/stop commands
(issued with edeliver task locally or application command on remote host).

```sh
$ ssh billing
$ prod && bin/billing ping
Node 'billing_prod@127.0.0.1' not responding to pings.
$ prod && bin/billing remote_console
Node billing_prod@127.0.0.1 is not running!
```

steps to reproduce:

- restart `prod` application - try to ping `stage` application

**solution**

see [Elixir - EPMD]({% post_url 2017-09-02-elixir-epmd %}).

## The task "phx.new" could not be found

```sh
$ mix phx.new billing --no-brunch
** (Mix) The task "phx.new" could not be found
```

**solution**

I guess this problem is related to using `asdf` to manage Elixir versions -
Phoenix has not been installed yet into new Elixir directory:

```sh
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
Are you sure you want to install "https://github.com/phoenixframework/archives/raw/master/phx_new.ez"? [Yn]
* creating /Users/tap/.asdf/installs/elixir/1.5.1/.mix/archives/phx_new
```

## all styles are gone

**solution**

I removed _priv/static/_ directory some time ago - restore it from Git repo
or copy from newly generated project:

```sh
$ mix phx.new hello --no-brunch
```

## [edeliver] Host key verification failed

```sh
$ mix deploy.stage
EDELIVER BILLING WITH UPDATE COMMAND
-----> Updating to revision dec86cd from branch master
-----> Building the release for the update
-----> Authorizing hosts
-----> Ensuring hosts are ready to accept git pushes
-----> Pushing new commits with git to: billing@billing
-----> Resetting remote hosts to dec86cdd704a1e88fd5fd0600bad4a425c81762e
-----> Cleaning generated files from last build
-----> Symlinking stage.secret.exs
-----> Fetching / Updating dependencies
using mix to fetch and update deps
rebar and rebar3 for mix was built already
* creating /home/billing/.mix/archives/hex-0.17.0
* Getting slime (git@github.com:slime-lang/slime.git)
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
** (Mix) Command "git --git-dir=.git fetch --force --quiet --progress" failed
```

**solution**

1. <https://github.com/docker-library/golang/issues/148#issuecomment-279459932>

use HTTPS instead of SSH to clone public repos in _mix.exs_:

```elixir
defp deps do
  [
    # ...
-   {:slime, git: "git@github.com:slime-lang/slime.git", override: true},
+   {:slime, git: "https://github.com/slime-lang/slime.git", override: true},
    # ...
  ]
end
```

## [distillery] "$@" -- "${1+$ARGS}"

systemd journal:

```
localhost systemd[1]: Started Phoenix server for billing.
localhost billing[1234]: "$@" -- "${1+$ARGS}"
```

**solution**

<https://github.com/bitwalker/distillery/issues/319#issuecomment-326661509>:

> It's put there when reattaching stdout to the tty after running post-start
> hooks, so it's normal. I'll see if I can silence it, but it's expected.
