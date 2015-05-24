---
layout:     post
title:      "Install Nginx side by side with Apache"
date:       2014-10-01
tags:
- Red Hat 6.4
- Nginx
---

Before we start you should make sure your openssl is updated. Run `rpm -q openssl`
and if this reports version `1.0.1e` and less than `1.0.1e-16.el6_5.4.0.1` then
you are vulnerable to heartbleed. One of the biggest security issues. Read
[Update openssl on Red Hat](update_open_ssl) before you continue. We also need
version `1.0.1` or higher to install Nginx.

First of you can check if Nginx is available on your system by running
`yum info nginx`. It should now show you the available package with its version
included, in my case `Version : 1.0.15`. If it's not available or if you get an
old version like me (the latest one is currently `1.6.2`) then you need to add
there official repo to yum. Create and open `/etc/yum.repos.d/nginx.repo` in
your favorite editor and add the config below.

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=0
enabled=1
```
If you run `yum info nginx` again you should now get `Version : 1.6.2` or later.
Then install it with `yum install nginx`.


[update_open_ssl]: the_url
