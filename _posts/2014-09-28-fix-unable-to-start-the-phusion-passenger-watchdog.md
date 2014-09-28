---
layout:     post
title:      "Fix: Unable to start the Phusion Passenger watchdog"
date:       2014-09-28
---

Does this look familiar for you? If it does, then you are on the right place.


```
# /var/log/httpd/error_log
[Wed Sep 24 18:21:20 2014] [notice] caught SIGTERM, shutting down
[Wed Sep 24 18:21:20 2014] [notice] SELinux policy enabled; httpd running as context unconfined_u:system_r:httpd_t:s0
[Wed Sep 24 18:21:20 2014] [notice] suEXEC mechanism enabled (wrapper: /usr/sbin/suexec)
[Wed Sep 24 18:21:20 2014] [error] *** Passenger could not be initialized because of this error: Unable to start the Phusion Passenger watchdog (/usr/local/lib/ruby/gems/2.1.0/gems/passenger-4.0.52/buildout/agents/PassengerWatchdog): Permission denied (errno=13)
```

I did get this error while configured a new server with CentOS 6.5, Apache and
Passenger for an Ruby on Rails application. If your setup looks different, then
you may not be able to do exactly like me. I did google allot on `Unable
to start the Phusion Passenger watchdog` and `Permission denied (errno=13)` but
without success. I first thought it had something to do with file permissions
but it wasn't. The real problem here are SELinux security policy. We do not want
to turn this one of because it actually are trying to do good for us. After all
it's there for our security.

We need to know which services that is prevented in order to start passenger and
allow only those. To do that we start by setting SELinux into permissive mode.
This will allow us to run all application without it being stoped by SELinux use
but it will log warnings which we will use to create rule and permit what we need.

Open `/etc/selinux/config` and change `SELINUX=enforcing` to `SELINUX=permissive`.

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#   enforcing - SELinux security policy is enforced.
#   permissive - SELinux prints warnings instead of enforcing.
#   disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
#   targeted - Only targeted network daemons are protected.
#   strict - Full SELinux protection.
SELINUXTYPE=targeted
```

Now you need to reboot your server in order for these settings to take action.

Start the processes needed for your application to run (mysql, redis and so on)
and then visit your page in the browser. It should now be working because
everything is permitted but will create warnings.

Now we can create our rules by running the command bellow.

```bash
grep httpd_t /var/log/audit/audit.log | audit2allow -m httpdrules > httpdrules.te
```

You can now also run `cat httpdrules.te` and make sure it doesn't contain anything
suspicious. If it looks ok then we can make it permanent by running

```bash
# notice the capitalized -M instead of -m that we used before
grep httpd_t /var/log/audit/audit.log | audit2allow -M httpdrules
semodule -i httpdrules.pp
```

Next step to se if it all is working is to change `permissive` back to `enforcing`
in our `/etc/selinux/config` file and reboot.

## Q & A
<dl>
<dt>It still doesn't work!</dt>
<dd>
If you get an similar error you can look at <a href="http://wiki.centos.org/HowTos/SELinux#head-aa437f65e1c7873cddbafd9e9a73bbf9d102c072">CentOS wiki</a>
how to customize these rules more. Else if you got another error this may just be one of a few errors in your stack. Google is your friend.
<dd>

<dt>How to install <code>audit2allow</code> if it's not available</dt>
<dd>
It comes with <code>policycoreutils-python</code> which is often installed with the system.
If this isn't on your system for any reason it can be installed with <code>yum install policycoreutils-python</code>.
</dd>

<dt>I don't use CentOS but i have the same error, will this work for me?</dt>
<dd>
This guide is written based on my experiences with CentOS, Apache, Passenger and
a Ruby on Rails application i tried to deploy. Some parts in this may still be
useful for you. But you will need to google a bit if your files don't look the same or if
they are not on the same location.
</dd>

<dt>I don't care about security, how to turn of SELinux completly?</dt>
<dd>
This is something you should only do if you not planing to go public with your
application. If this is true then just change your <code>SELINUX</code> option
in <code>/etc/selinux/config</code> to <code>disabled</code> and reboot.
</dd>
</dl>
