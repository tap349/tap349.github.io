---
layout: post
title: GitHub Pages - Custom domain
date: 2018-06-19 12:28:25 +0300
access: private
comments: true
categories: [dns]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://help.github.com/articles/quick-start-setting-up-a-custom-domain/>

- `DNS provider` = `domain registrar` = `DNS host` (say, `Namecheap`)
- `root domain` = `apex domain` (say, `example.com`)

> <https://help.github.com/articles/setting-up-a-custom-subdomain/>
>
> You can set up a custom subdomain, such as blog.example.com, by creating
> a CNAME record through your DNS provider.

register root domain with your DNS provider
-------------------------------------------

I want to use subdomain `blog.tap349.com` of my root domain `tap349.com` as
a custom domain for my GitHub Pages site (<http://tap349.github.io>).

add custom domain in repo settings on GitHub
--------------------------------------------

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>

this will create _CNAME_ file with a custom domain in the root of your repo:

```
blog.tap349.com
```

add CNAME record for custom domain on DigitalOcean
--------------------------------------------------

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>
2. <https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-dns#cname-records>

- `HOSTNAME`: `blog.tap349.com`
- `IS AN ALIAS OF`: `tap349.github.io`

this will make DNS lookups for <http://blog.tap349.com> to redirect to
<http://tap349.github.io>.

update your DNS provider nameservers
------------------------------------

1. <https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars>

> To use DigitalOcean DNS, you'll need to update the nameservers used by your
> domain registrar to DigitalOcean's nameservers instead.

enter DigitalOcean nameservers in DNS settings of your DNS provider
(it might be necessary to select your root domain first):

- `ns1.digitalocean.com`
- `ns2.digitalocean.com`
- `ns3.digitalocean.com`

check custom domain is set up correctly
---------------------------------------

1. <https://help.github.com/articles/setting-up-a-custom-subdomain/>

```
$ dig blog.tap349.com +nostats +nocomments +nocmd
; <<>> DiG 9.10.6 <<>> blog.tap349.com +nostats +nocomments +nocmd
;; global options: +cmd
;blog.tap349.com.		IN	A
blog.tap349.com.	43200	IN	CNAME	tap349.github.io.
tap349.github.io.	3600	IN	CNAME	sni.github.map.fastly.net.
...
```
