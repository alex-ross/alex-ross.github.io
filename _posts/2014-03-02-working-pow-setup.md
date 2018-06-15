---
layout: post
title:  "Get pow running with ZSH and Rbenv"
date:   2014-03-02
categories: pow rbenv
tags:
- pow
- rbenv
---

Usually I'm using `rails s` with thin or webbrick when developing web
applications because I've had lots of issues with pow and file uploads. But when
I'm working on multiply projects within same day it's really handy to have pow
installed.

*Hey wait, what is pow?* Pow is according to there own website "a zero-config
Rack server for Mac OS X". But it's still needs some configuration... It's
really great when working on multiply rack applications. It will boot up the
apps you visit in the browser and have support for subdomains so you don't have
to mix with your DNS or hosts file. You can read more about pow on there website
at [pow.cx](http://pow.cx/).

In my case the zero-config part did not apply. Whitout any configurations this
is what I see.
![LoadError: cannot load such file -- bundler/setup][bundler_setup_error]

Pow has a good [troubleshooting][pow_throuble] with lots of examples for
similar problems.

Because I'm using both ZSH and Rbenv I listen to there advise and put this
content in my `~/.powconfig`.
{% highlight bash %}
export HOME=/Users/ross
export PATH="/usr/local/opt/rbenv/shims:/usr/local/opt/rbenv/bin:$PATH"
{% endhighlight %}

Sadly I still have the exact same result as before.

I'm also adding `eval "$(rbenv init -)"` at the bottom of the file which I read
in a issue they linked to in the troubleshooting. Now it tells me that I haven't
installed the correct ruby version used for this project which I know I have
installed.

![Error: rbenv: version `1.9.3-p327' is not installed`][wrong_ruby_version]

From here I could guess that it's not finding the correct path to my rbenv. When
I run `echo $(rbenv root)` in my terminal and found out that my root path is
`/usr/local/var/rbenv`. I'm also know that rbenv uses `RBENV_ROOT` and `echo
$RBENV_ROOT` gives the same path in return. So my problem may be that I have
multiply versions of rbenv installed. Next step will be to tell pow to use the
same version I'm using in my terminal. Just changing the PATH from
`/usr/local/opt/rbenv` to `/usr/local/var/rbenv` didn't work but when I also set
`RBENV_ROOT` to `/usr/local/var/rbenv` it did. Yey!!!

## My `~/.powconfig` file
{% highlight bash %}
export HOME=/Users/ross
export RBENV_ROOT=/usr/local/var/rbenv
export PATH=$RBENV_ROOT/shims:$RBENV_ROOT/bin:$PATH
eval "$(rbenv init -)"
{% endhighlight %}

[bundler_setup_error]: {{site.url}}/img/posts/2014-03-02/bundler-setup.png
[wrong_ruby_version]: {{site.url}}/img/posts/2014-03-02/wrong-ruby-version.png
[pow_throuble]: https://github.com/basecamp/pow/wiki/Troubleshooting#rbenv
