---
layout: post
title: GitHub Pages - Custom Domain
date: 2018-06-19 12:28:25 +0300
access: public
comments: true
categories: [dns]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>DNS provider</dt>
  <dd>
    company providing DNS servers (nameservers) - it might be your domain
    registrar or entirely different company (say, DigitalOcean)
  </dd>

  <dt>root domain</dt>
  <dd>= apex domain (say, example.com)</dd>
</dl>

<hr>

1. <https://www.namecheap.com/support/knowledgebase/article.aspx/766/10/what-is-dns-server-name-server>
2. <https://help.github.com/articles/using-a-custom-domain-with-github-pages/>
3. <https://help.github.com/articles/quick-start-setting-up-a-custom-domain/>

- `A` DNS records are for IPv4 only
- `AAAA` DNS records are for IPv6 only

> <https://help.github.com/articles/setting-up-a-custom-subdomain/>
>
> You can set up a custom subdomain, such as blog.example.com, by creating a
> CNAME record through your DNS provider.

## register root domain

I want to use subdomain `blog.tap349.com` of my root domain `tap349.com` as a
custom domain for my blog on GitHub Pages (<http://tap349.github.io>) - I will
refer to it as a custom blog domain hereafter to avoid ambiguity.

## add custom blog domain in repo settings on GitHub

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>

this will create _CNAME_ file with a custom blog domain in the root of blog repo
(in a separate commit):

```
blog.tap349.com
```

## update nameservers used by your domain registrar

1. <https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars>

> To use DigitalOcean DNS, you'll need to update the nameservers used by your
> domain registrar to DigitalOcean's nameservers instead.

enter DigitalOcean nameservers in DNS settings of your domain registrar (it
might be necessary to select your root domain first):

- `ns1.digitalocean.com`
- `ns2.digitalocean.com`
- `ns3.digitalocean.com`

> It will take some time for the name server changes to propagate after you've
> saved them. During this time, the domain registrar communicates the changes
> you've made with your ISP (Internet Service Provider). In turn, your ISP
> caches the new nameservers to ensure quick site connections. This process
> usually takes about 30 minutes, but could take up to a few hours depending on
> your registrar and your ISP's communication methods.

## add CNAME record for custom blog domain

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>
2. <https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-dns#cname-records>

CNAME record must be created through your current DNS provider.

| DigitalOcean                                |
| ------------------------------------------- |
| `Create ▼` (top right menu) → `Domains/DNS` |
| `<YOUR_DOMAIN>` → `CNAME` (tab)             |

- `HOSTNAME`: `blog.tap349.com`
- `IS AN ALIAS OF`: `tap349.github.io`

this will make DNS lookups for <http://blog.tap349.com> to redirect to
<http://tap349.github.io>.

at the same time if you open <http://tap349.github.io> in browser it will
substitute URL for <http://blog.tap349.com>.

## check custom blog domain is set up correctly

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>

{% raw %}

```
$ dig blog.tap349.com +nostats +nocomments +nocmd
; <<>> DiG 9.10.6 <<>> blog.tap349.com +nostats +nocomments +nocmd
;; global options: +cmd
;blog.tap349.com.		IN	A
blog.tap349.com.	43200	IN	CNAME	tap349.github.io.
tap349.github.io.	3600	IN	CNAME	sni.github.map.fastly.net.
...
```

{% endraw %}

## _[OPTIONAL]_ use root domain as a custom blog domain

1. <https://help.github.com/articles/setting-up-an-apex-domain/>

NOTE: you cannot have multiple custom blog domains.
