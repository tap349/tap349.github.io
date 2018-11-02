---
layout: post
title: SSH - Public Key Authentication
date: 2016-03-04 16:13:00 +0300
access: public
comments: true
categories: [ssh]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>PKA</dt>
  <dd>public key authentication</dd>

  <dt>authorized key</dt>
  <dd>public key permitted for logging in</dd>

  <dt>identity key, identity</dt>
  <dd>private key</dd>
</dl>

<hr>

1. <https://www.ssh.com/ssh/public-key-authentication>
2. <https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.foto100/pkauth.htm>
3. <https://wiki.archlinux.org/index.php/SSH_keys>

description
-----------

> <https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.foto100/pkauth.htm>
>
> When the user logs in, ssh tells the server which key pair it would like
> to use for authentication. The client proves that it has access to the
> private key and the server checks that the corresponding public key is
> authorized to accept the account.

SSH agent
---------

> man 1 ssh-agent
>
> The agent will never send a private key over its request channel. Instead,
> operations that require a private key will be performed by the agent, and
> the result will be returned to the requester. This way, private keys are
> not exposed to clients using the agent.

each host entry in SSH config (_~/.ssh/config_) has `IdentityFile` option
which specifies the path to identity file for this host.

SSH client uses SSH config to decide which identity to use by looking up
required host in SSH config and reading its `IdentityFile` option.

SSH client then asks SSH agent to provide this identity to authenticate
login to remote server - AFAIU identity is identified either by its file
path or corresponding public key (both are shown in `ssh-add -L` output).

### manual setup

- start SSH agent

  start SSH agent and export required environment variables:

  ```sh
  $ eval `ssh-agent -s`
  ```

  SSH client uses these variables to establish connection to SSH agent.

  > <http://www.snailbook.com/faq/about-agent.auto.html>
  >
  > Some people express irritation over this seemingly convoluted procedure,
  > and wonder why they can't just run ssh-agent and be done with it.
  >
  > In Unix, there is no way for a process to directly change the environment
  > of other existing processes; it can only change its own environment, and
  > those of child processes it starts. Thus, running ssh-agent cannot affect
  > the environment of the shell which starts the agent.
  >
  > Having the agent print out shell commands which can be easily executed to
  > set the variables, is as convenient as it gets.

- add identities to SSH agent

  1. <https://www.ssh.com/ssh/config>

  > man 1 ssh-agent
  >
  > The agent initially does not have any private keys. Keys are added using
  > ssh(1) (see AddKeysToAgent in ssh_config(5) for details) or ssh-add(1).
  > Multiple identities may be stored in ssh-agent concurrently and ssh(1)
  > will automatically use them if present.

  add default identities to SSH agent:

  ```sh
  $ ssh-add
  Identity added: /Users/tap/.ssh/id_rsa (/Users/tap/.ssh/id_rsa)
  ```

  > man 1 ssh-add
  >
  > When run without arguments, it adds the files ~/.ssh/id_rsa,
  > ~/.ssh/id_dsa, ~/.ssh/id_ecdsa, and ~/.ssh/id_ed25519.

- list loaded identities

  list public keys of all loaded identities:

  ```sh
  $ ssh-add -L
  ssh-rsa AAAAB... /Users/tap/.ssh/id_rsa
  ```

  list fingerprints of all loaded identities:

  ```sh
  $ ssh-add -l
  2048 SHA256:iCeg1... /Users/tap/.ssh/id_rsa (RSA)
  ```

- set up agent forwarding

  > <https://www.ssh.com/ssh/agent#sec-SSH-Agent-Forwarding>
  >
  > Furthermore, the SSH protocol implements agent forwarding, a mechanism
  > whereby an SSH client allows an SSH server to use the local ssh-agent on
  > the server the user logs into, as if it was local there.
  >
  > When the user uses an SSH client on the server, the client will try to
  > contact the agent implemented by the server, and the server then forwards
  > the request to the client that originally contacted the server, which
  > further forwards it to the local agent.

  ```diff
    # ~/.ssh/config

    Host lain
      User lain
      Hostname 172.104.X.X
      IdentityFile ~/.ssh/id_rsa
  +   ForwardAgent yes
  ```

### automatic setup

1. <https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/ssh-agent>

`ssh-agent` Zsh plugin can be used to start SSH agent and add identities
automatically:

```sh
# _~/.zshrc_

plugins(ssh-agent ...)
```

only default identities are added by default (it's possible to add other
identities though - see documentation).
