---
layout: post
title:  "Create an Ubuntu Application Indicator in Python: step-by-step guide"
date:   2014-09-13
categories: appindicator
---

When implementing the [Vagrant AppIndicator for Ubuntu](https://github.com/candidtim/vagrant-appindicator), I really 
lacked the documentation on this subject. I've found the 
[official documentation](https://unity.ubuntu.com/projects/appindicators/) too sparse and having broken links (there is
still one broken by the time of this writing) and the only way to proceed was to experiment a lot and dig into
open-source implementations of a couple of other application indicators. So, I've decided to release this guide, 
hopefully comprehensive enough and useful. Here I show, step-by-step, how the AppIndicator could be implemented in 
Python. I try to keep it short and concrete, yet understandable.

## Minimal set-up - the very basic indicator

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

Few notes on it:

1. First, we import [Gtk](http://python-gtk-3-tutorial.readthedocs.org/en/latest/#) and 
   [AppIndicator](http://developer.ubuntu.com/api/devel/ubuntu-13.10/python/AppIndicator3-0.1.html) basic
   implementation. Thing to note here is that in some older documentation you may find importing `gtk` module - that 
   would be an older Gtk 2, while we use Gtk+ 3 here.

2. A most simple AppIndicator, in order to be actually displayed, would need to be activated (`set_status(...ACTIVE)`),
   and would need to have a menu associated with it. If any of these conditions is not met, the application will start,
   but there will be no AppIndicator displayed though. Thus, after instantiating the one, we set it up correctly.

   > A note on AppIndicator constructor: it accepts a **unique** indicator name (so think of a name no one else would 
   > use), an icon name (more on it later, let's put any string here for now) and an indicator category. You can `dir()`
   > all categories from `appindicator.IndicatorCategory` to see which are available. The category typically impacts 
   > where in the system tray the AppIndicator is placed - that is, the ordering between AppIndicators. You may want
   > to experiment with different categories to see what the result is.

3. Lastly, `gtk.main()` starts the Gtk endless loop. This function call will not quit until the Gtk application ends
   itself. Thus, in the case of the code above - it never ends.

If you run this code now from Python REPL or, run [this script](https://gist.github.com/candidtim/c943835a9742f5021eeb)
- you should now see the AppIndicator in the "system tray", with a standard "no icon" icon.

![Minimalist Ubuntu AppIndicator](/assets/myappindicator_v1.png)

This is our basic starting point. Let's start improving it now.

> [Full source code at this point](https://gist.github.com/candidtim/c943835a9742f5021eeb)

## Can't stop?

First thing you'll notice - there is no "normal" way to stop the AppIndicator. We didn't add a call to stop the Gtk 
main loop anywhere, and even `Ctrl+C` doesn't really work. For now, the only thing you can do is to find the id of the 
process and kill it. Or, simply close the terminal you've started the application in.

First quick win would be to fix "Ctrl+C" behavior. Add:

{% highlight python %}
import signal
{% endhighlight %}

and the following line into the main method, anywhere before the Gtk main loop is started:

{% highlight python %}
signal.signal(signal.SIGINT, signal.SIG_DFL)
{% endhighlight %}

This will make the AppIndicator reacting normally to the "Ctrl+C" in the terminal - at least it can be stopped now. 

> This [sets the handler](https://docs.python.org/2/library/signal.html#signal.signal) for "INT" signal 
> processing - the one issued by the OS when "Ctrl+C" is typed. The handler we assign to it is the 
> ["default"](https://docs.python.org/2/library/signal.html#signal.SIG_DFL) handler, which, in case of the interrupt
> signal, is to stop execution.

However, for the end user we would certainly want to add a more convenient way to stop it - which would typically be a
dedicated menu item in the indicator's main menu. Let's add it now.

> [Full source code at this point](https://gist.github.com/candidtim/2cd33ad40016b918ecf9)

## Add menu items

AppIndicator is built on Gtk, so there is nothing specific in the way you create a menu. Gtk+ 3 has a 
[great tutorial](http://python-gtk-3-tutorial.readthedocs.org/en/latest/#). Below is the example of the menu for out 
AppIndicator. Let's add "Quit" menu item.

Add two new functions:

{% highlight python %}
def build_menu():
    menu = gtk.Menu()
    item_quit = gtk.MenuItem('Quit')
    item_quit.connect('activate', quit)
    menu.append(item_quit)
    menu.show_all()
    return menu
 
def quit(source):
    gtk.main_quit()
{% endhighlight %}

And, replace `indicator.set_menu(gtk.Menu())` with `indicator.set_menu(build_menu())`, to assign the new menu to an 
indicator. Not digging into Gtk details, few notes on this however:

1. Do not forget to call `menu.show_all()`, or the menu will not contain the new items added to it.

2. Action listener assigned to the menu item receives a single argument - an event source, which in this case is the
   menu item itself. We don't need to use it in this implementation so could have replaced the argument name with `_`.

3. Calling `gtk.main_quit()` stops the Gtk main loop, effectively making the previous call to `gtk.main()` to 
   return, and thus, our `main()` function to return and application to stop.

Now if you run it and click on an indicator icon - there will be "Quit" menu item, which, when clicked should stop
the indicator application. 

But, the indicator itself still looks quite ugly. Let's find a better icon for it now.

> [Full source code at this point](https://gist.github.com/candidtim/4705c55d41ae982ee60e)

## Custom icon 

There are two options for an indicator icon: use one from the icon library installed with the Gtk, or, use a custom
one. To use the icon from the Gtk library, simply, give its name to the AppIndicator constructor. For example, following
change:

{% highlight python %}
indicator = appindicator.Indicator.new(APPINDICATOR_ID, gtk.STOCK_INFO, appindicator.IndicatorCategory.SYSTEM_SERVICES)
{% endhighlight %}

Will result in:

![Ubuntu AppIndicator with Gtk icon](/assets/myappindicator_v2.png)

> Prebuilt Gtk icons depend on the Gtk distribution, but there are few which normally present in every distribution. You
> can search the documentation to find the list of all icons, like 
> [this one](http://www.pygtk.org/pygtk2reference/gtk-stock-items.html) for example.

Although probably little better than "no icon" icon, yet this doesn't look unique. Alternatively, you can use a custom
image as an icon. I think that the best choice for an icon is the `SVG` image, as it will scale nicely if the UI theme
requires system tray icons to have size different from the one you've tested on. 

Create your icon, or download one, or use [this sample icon](/assets/sample_icon.svg) I've created for this guide. Save
it somewhere on disk, and use the path to an icon instead of the icon name when constructing the AppIndicator. Path can
be absolute or relative, so, if you save an icon into a file `sample_icon.svg` in the same directory as the AppIndicator
python program, update the call to the constructor as follows:

{% highlight python %}
indicator = appindicator.Indicator.new(APPINDICATOR_ID, 'sample_icon.svg', appindicator.IndicatorCategory.SYSTEM_SERVICES)
{% endhighlight %}

Run, and now it should look like this:

![Ubuntu AppIndicator with custom icon](/assets/myappindicator_v3.png)

Nice! Note that an icon can also be changed while the indicator is running, with `indicator.set_icon()` method.

> [Full source code at this point](https://gist.github.com/candidtim/7290a1ad6e465d680b68)

## Showing balloon (bubble) notifications

With all above we have now a ready-to-use base for an AppIndicator. Now, the AppIndicator doesn't do that much yet. 
As an example, let's add some behavior to it - for example, give us some nerdy Chuck Norris joke from 
[The Internet Chuck Norris Database](http://www.icndb.com/api/). We would display jokes in a balloon notification 
typically popping out in upper right corner of the screen in Ubuntu.

We would need 3 elements here. First, let's add a function to fetch a joke:

{% highlight python %}
import json
from urllib2 import Request, urlopen, URLError

# ...

def fetch_joke():
    request = Request('http://api.icndb.com/jokes/random?limitTo=[nerdy]')
    response = urlopen(request)
    joke = json.loads(response.read())['value']['joke']
    return joke
{% endhighlight %}

now, add a menu item and an action listener:

{% highlight python %}
def build_menu():
	# ...
    item_joke = gtk.MenuItem('Joke')
    item_joke.connect('activate', joke)
    menu.append(item_joke)
    # ...

def joke(_):
    # ...
{% endhighlight %}

and, finally, show the joke in a balloon notification:

{% highlight python %}
from gi.repository import Notify as notify

# ...

def main():
    # ...
    notify.init(APPINDICATOR_ID)
    gtk.main()

# ...

def joke(_):
    notify.Notification.new("<b>Joke</b>", fetch_joke(), None).show()

def quit(_):
    notify.uninit()
    gtk.main_quit()
{% endhighlight %}

Again, few notes:

1. To show notifications we use [notify API](https://notify2.readthedocs.org/en/latest/) which we first import and 
   initialize with the unique application name - in our case could be the same as the one used for an indicator itself.

2. To show a notification we construct a `Notification` instance and `show()` it. There is no 3rd argument in the code 
   above, which is a path to an icon. If you pass `None` as above, there will simply no icon.

3. Be nice to Gtk and `uninit()` notify API before quitting the application

Now, if you run the AppIndicator at this point, clicking on its "Joke" menu should display a fun balloon notification:

![Joking Ubuntu AppIndicator](/assets/myappindicator_v4.png)

That's it! I take no more your attention, and hope this sample AppIndicator could serve a good base for a new indicator implementation ;)

You can find [here the source code of this implementation](https://gist.github.com/candidtim/5663cc76aa329b2ddfb5) and
use it as you want (consider it as MIT). Or you can also check out the 
[Vagrant AppIndicator for Ubuntu](https://github.com/candidtim/vagrant-appindicator) as a more real-world example which
also provides theming support (dark & light UI themes) and a way to distribute an AppIndicator in a Python package.