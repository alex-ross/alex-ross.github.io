---
layout: post
title:  "How to SSH through SSH proxy"
date:   2015-07-27 17:58
tags:
- ssh
- proxy
---

At our company we do not want any outsiders to access our servers through SSH.
Often we prevent logins to root and prevent logins with password. But to be even
more secure we started to only allow logins from the IP addresses at our office.

At our office we have an Mac Mini that is open for SSH to all employees. Here we
can have a main list of allowed SSH keys and IP addresses and everyone can SSH
Proxy through this one. This makes it a lot easier for us to manage access.

## So how do you SSH through another server

Lets say that our computer we want to use as an SSH Proxy has the domain
"proxy.example.com" and the server we will ssh to has "database.example.com". Lets
also assume that the user for both servers in my case are "aross".

SSH has a lot of options we can use. The one we will use is called
`ProxyCommand`. We will first give it user and host to the proxy server followed
by the `-W`-flag where we tell it to which host and port we want to bind this
proxy to. Instead of writing host and port in this case we can use the
placeholders `%h` for **h**ost and `%p` for **p**ort and the ssh service will
figure it all out by itself.

The command will look like this

``` bash
ssh -o ProxyCommand='ssh aross@proxy.example.com -W %h:%p' aross@database.example.com
```

Thats basically it. If you do this allot you could simplify the process by
editing your `~/.ssh/config` file.

Add this to `~/.ssh/config/`

```
Host officeproxy
  User aross
  HostName proxy.example.com

Host database.example.com
  HostName database.example.com
  ProxyCommand ssh officeproxy -W %h:%p
```

Now when you do `ssh aross@database.example.com` SSH will set up the proxy for
you according to the config file.


