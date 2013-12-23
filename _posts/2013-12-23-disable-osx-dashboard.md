---
layout: post
title:  "Disable OS X Dashboard"
date:   2013-12-23 16:00
categories: apple osx mavericks
---

If you are like me and never uses Dashboard on your mac. Why should you still
have it there?

I found out that it can be killed and disabled.

First you need to open up your terminal and type:
{% highlight bash %}
defaults write com.apple.dashboard mcx-disabled -boolean true
{% endhighlight %}
Then you just have to relaunch the dock by typing:
{% highlight bash %}
killall Dock
{% endhighlight %}

Now your done!


Source: [Techradar][1]

[1]: http://www.techradar.com/news/software/operating-systems/20-os-x-mavericks-tips-and-tricks-1190830
