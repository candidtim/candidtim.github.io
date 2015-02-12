---
layout: post
title:  "Create an Ubuntu Application Indicator in Python: step-by-step guide"
date:   2014-09-13
categories: appindicator
---

When implementing the [Vagrant AppIndicator for Ubuntu](https://github.com/candidtim/vagrant-appindicator), I really 
lacked the documentation on this subject. I've found the 
[official documentaiton](https://unity.ubuntu.com/projects/appindicators/) too sparse and having broken links (there is
still one broken by the time of this writing) and the only way to proceed was to experiment a lot and dig into
open-soruce implementations of a couple of other application indicators. So, I've decided to release this guide, 
hopefully comprehensive enough and useful. Here I show, step-by-step, how the AppIndicator could be implemented in 
Python.

I try to keep it short and concrete, yet understandible. Where a more detailed explanation is needed I will move it to
a dedicated section in the end of the post.

## Minimal set-up - basic indicator with a sample icon

The very basic AppIndicator in Python would be like this:

{% highlight python %}
from gi.repository import Gtk as gtk
from gi.repository import AppIndicator3 as appindicator

APPINDICATOR_ID = 'myappindicator'

def main():
    indicator = appindicator.Indicator.new(APPINDICATOR_ID, 'whatever', appindicator.IndicatorCategory.SYSTEM_SERVICES)
    indicator.set_status(appindicator.IndicatorStatus.ACTIVE)
    indicator.set_menu(gtk.Menu())
    gtk.main()

if __name__ == "__main__":
    main()
{% endhighlight %}

First, we import [Gtk+ 3](http://python-gtk-3-tutorial.readthedocs.org/en/latest/#) and 
[AppIndicator](http://developer.ubuntu.com/api/devel/ubuntu-13.10/python/AppIndicator3-0.1.html) basic implementation. 
Thing to note here is that in some older documentation you may find importing `gtk` module - that would be an older 
Gtk 2.

A most simple AppIndicator, in order to be actually displayed, would need to be activated (`set_status(...ACTIVE)`), and
would need to have a menu assiciated with it. If any of these conditions is not met, the applicatoin will start, but
there will be no AppIndicator displayed though. Thus, after instantiaing the one, we set it up correctly.

> A note on AppIndicator constructor: it accepts a *unique* indicator name (so think of a name no one else would use), 
> an icon name (more on it later, let's put any string here for now) and an indicator category. 

> You can `dir()` all categoreis from `appindicator.IndicatorCategory` to see which are available. The category typically
> impacts where in the system tray the AppIndicator is placed - that is, the ordering between AppIndicators. You may want
> to experiment wth different categories to see what the result it.

Lastly, `gtk.main()` starts the Gtk endless loop. This function call will not quit until the Gtk application ends itself.
Thus, in the case of the code above - it never ends.

If you run this code now from Python REPL or create a script and run it - you should now see the AppIndicator in the
"system tray", with a standard "no icon" icon.

	TODO: image here

This is our basic starting point. Let's improve it now.

	TODO: full source code at this point

## Can't stop?

First thing you will notice - there is no "normal" way to stop the AppIndicator. We didn't add a call to stop the Gtk 
main loop anywhere, and even `Ctrl+C` doesn't really work. For now, the only thing you can do is to find the prcess id 
and kill it. Or, simply close the terminal you've started the application in.

First quick win would be to fix "Ctrl+C" behaviour. Add:

{% highlight python %}
import signal
{% endhighlight %}

and the following line into the main method, anywhere before the Gtk main loop is started:

{% highlight python %}
signal.signal(signal.SIGINT, signal.SIG_DFL)
{% endhighlight %}

This will make the AppIndicator reacting normally to the "Ctrl+C" in the terminal - at least it can be stopped now. 

> This line of code [sets the handler](https://docs.python.org/2/library/signal.html#signal.signal) for "INT" signal 
> processing - the one issued by the OS when "Ctrl+C" is typed - and the handler is the 
> ["default"](https://docs.python.org/2/library/signal.html#signal.SIG_DFL) one, which, in case of the interrupt signal,
> is to stop execution.

However, for the end user we would certianly want to add a more convenient way to stop it - which would typically be a
dedicated menu item in the indicator's main menu. Let's add it now.

	TODO: full source code at this point

## Add menu items

AppIndicator is built on Gtk, so there is nothing specific in the way you create a menu. Here I will not dive into Gtk
itself, as there are really good tutorials on it, like 
[this one](http://python-gtk-3-tutorial.readthedocs.org/en/latest/#), for example. The simple way to create a menu for
an AppIndicator is below. Let's add two menu items - as decided one is "Quit", and other sample one we'll enrich later.

Instead of setting an empty menu, as our initial implementation does, let's build it first:

	TODO

## Custom icon 

...

## Showing baloon notifications

{% highlight python %}
from gi.repository import Notify as notify

init first
{% endhighlight %}

## Testing the AppIndicator

	TODO

## Distributing the AppIndicator - why not Python package?

Certainly the best option to distribute an AppIndicator for Ubuntu would be to create a Debian package for it. I've 
found however creating the `deb` package quite cumbersome, and, I decided to not to go with this solution - mosty 
because I would need to set-up a ppa repository and maintain it.

As an alternative, as long as the AppIndicator is implemented in Python, it can be distributed in a Python package. Now,
it could be installed with `pip` directly from source code, like this:

    pip install git+https://example.com/myappindicator.git

There is an obvious disadvantage of this approach howeever - no automatic update would be possible when a newer version
of the AppIndicator is released. I think thus, that building a Debian package would be a better option, but if you're
still interested in the Python pacakging option, there it is:

	TODO

## Tips and Tricks - identify main colors of current Ubuntu theme

Imagine you'd like your AppIndicator having different icons depending on the UI theme in Ubuntu. For example, in darker
themese the icon can be lighter, and vice versa. But, how to identify current Ubuntu theme?

There is an API you could normally use to identify active UI theme name, but there is one disadvantage with it - in your
application you would need to know the names of all themes your users might use, in order to assign the right icon. 

...