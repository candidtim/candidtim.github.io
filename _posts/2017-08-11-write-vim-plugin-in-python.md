---
layout: post
title:  "Writing Vim plugin in Python"
date:   2017-08-11
categories: vim
---

> A draft of this post was originally published by accident. It apparently
> attracted some interest and, although the post was never complete, all code
> was working by the time it was published. As such this post is kept published
> as-is, in the hope that it still serves its purpose.

Writing a Vim script for own use is easy. Writing a plugin, compatible with pathogen, Vundle or other, and making
it top quality user experience is just a bit more complicated. But is still easy and fun. Here I show just how
to do so.

Only prerequisites are: Vim, some Python knowledge and lots of curiosity (more on that later). Some sample commands here
are for `bash`, so, Linux or Mac. You will need to adjust a bit if you use Windows, but the general idea and all the
rest is same in any case.

Let's go!


## Vim Python support

There are many ways to create a Vim plugin. Classic one - use VimL. Or you can also use Lua. Or Python. This particular
guide uses Python, and Python may be a great language to write a plugin for Vim because:

 - it is "*natively*" supported by Vim
 - you most likely already know it, in contrast to VimL
 - and it simple; you know, in contract to VimL

Certainly, a plugin written in Python will only run in Vim compiled with Python support. Vim's default distribution is
compiled with Python support, and nowadays finding the opposite is actually harder. There is also a number of widely
used Vim plugins written in Python and you shouldn't worry about Python support - it is not going anywhere.

To make sure that your Vim has Python support, run `vim --version`, and look for a line marked `+python` or `+python3`.
Note that all code below is designed for Python 2 (`+python`) which is how Vim is distributed by default. If your
Vim uses Python 3 (`+python3`) - you will need to update the source code accordingly.


## Principles and minimal template

Vim plugins actually *have to* be written in VimL and not in Python. Good news is that Vim plugin can execute arbitrary
Python scripts from withing VimL code. With this knowledge, basic idea of the plugin is to:

 - create a wrapper script in VimL
 - which will declare Vim commands
 - and import and run Python code
 - while latter implements those commands

Before going into Python code, let's prepare the basic project structure, development environment, and ensure that our
plugin is ready for plugin managers.


### Plugin structure

If we want our plugin to work with Vim plugin managers, like pathogen, Vundle and many others, it needs to follow
some basic structure:

{% highlight bash %}
sampleplugin/
├── doc/
│   └── sampleplugin.doc
└── plugin/
    └── sampleplugin.vim
{% endhighlight %}

This is self-explanatory. It is a good idea to provide an integrated documentation for a plugin, and
we will addres this later on. If we are to publish the plugin, say, on GitHub, it makes sense to also add two more
files:

{% highlight bash %}
sampleplugin/
├── ...
├── LICENSE
└── README
{% endhighlight %}

Once our project structure is ready, let's try and install it.


### Development process and our first Vim command

Let's configure the development environment at once, so that we can test and run the plugin in a Vim instance
regularly. How this set-up is made depends largely on the plugin manager you use with Vim.

Some plugin managers require all plugins to be installed under same root directory, which for most users is
`~/.vim/bundle`. If you are concerned, and don't want to change your plugins root directory, you can create a symbolic
link from your source code:

{% highlight bash %}
$ cd ~/.vim/bundle
$ ln -s ~/src/sampleplugin sampleplugin
{% endhighlight %}
Check you Vim's plugin manager documentation on how to declare and load the plugin. For example, I use
[Vundle](https://github.com/VundleVim/Vundle.vim), my plugin source code is in `~/src/sampleplugin`, and thus I have
following in my `~/.vimrc`:

{% highlight VimL %}
Plugin 'file:///home/candidtim/src/sampleplugin'
{% endhighlight %}


Now, let's make sure this actually works. Let's add following content to `sampleplugin.vim`:

{% highlight VimL %}
echo "It worked!"
{% endhighlight %}

And start new Vim instance where we will test the plugin. Upon startup you should see "hello vim!" printed out
in the terminal. It worked!

> If at this point it doesn't work, try to load the plugin manually. For this, execute following command from Vim:
> `:source ~/.vim/bundle/sampleplugin/plugin/sampleplugin.vim`. Now, if this finally works, it means that your plugin
> manager doesn't load the plugin automatically on Vim startup - refer to your plugin manager documentation to find out
> how to configure it correctly. If however this doesn't work either - Vim should normally print out an error message,
> which should give you a better idea. Most likely you need to check that file actually exists and symbolic link works
as expected, and that file content (syntax) is correct.

All set! Let's write some Python!


## Use Python in Vim plugin

As noted above, the idea now is to execute Python code from VimL. VimL exposes specific syntax for this. Let's change
our plugin source to the following:

{% highlight VimL %}
python << EOF
    print "Hello from Vim's Python!"
EOF
{% endhighlight %}

(Re-)start test Vim instance and you should see the new message.

Now, I don't mind writing few simple commands inline like this, but our actual goal is to make Python code to live in
Python source files, and VimL code in .vim source files. So, let's actually make Vim "import" our code from Python
source files. Change the code to:

{% highlight VimL linenos %}
let s:plugin_root_dir = fnamemodify(resolve(expand('<sfile>:p')), ':h')

python << EOF
import sys
from os.path import normpath, join
import vim
plugin_root_dir = vim.eval('s:plugin_root_dir')
python_root_dir = normpath(join(plugin_root_dir, '..', 'python'))
sys.path.insert(0, python_root_dir)
import sample
EOF
{% endhighlight %}

Vim doesn't know where your Python plugin code lives, so if we are to import it, we need to add its root directory
to `sys.path` in the interpreter running inside Vim. For this:

 - (1) we first save plugin's directory path into a local variable in plugin's Vim script
 - (7) then acces its value from within Python script
 - (8) use it to build the path to the directory where our Python code lives
 - (9) and finally add it to `sys.path`
 - (10) so that we can now import our Python module

To extract value from Vim's `plugin_root_dir` variable we use `vim` Python module. This is available inside Vim
and provides an interface to the Vim environment. We will revist this in details later.

Now, let's actually add this Python code we talk about. Let's add a file:

{% highlight bash %}
$ touch /sampleplugin/python/sample.py
{% endhighlight %}

With following content:

{% highlight python %}
print "Hello from Python source code!"
{% endhighlight %}

Restart test Vim instance, see the new message, all done!


## Declare commands and implement them in Python

Now, you likely want to add some commands to the Plugin, or it risks to not to be very useful. Let's implement a
simple command which would print out the country you are in, based on your IP. I mean, why not?

Let's implement it first:

{% highlight python %}
import urllib, urllib.request
import json
import vim

def _get(url):
    return urllib.request.urlopen(url, None, 5).read().strip().decode()

def _get_country():
    try:
        ip = _get('http://ipinfo.io/ip')
        json_location_data = _get('http://api.ip2country.info/ip?%s' % ip)
        location_data = json.loads(json_location_data)
        return location_data['countryName']
    except Exception as e:
        print 'Error in sample plugin (%s)' % e.msg

def print_country():
    print 'You seem to be in %s' % _get_country()
{% endhighlight %}

Now, the buty of this implementation is in that it is plain Python code. You can test and debug it outside Vim with
whatever tools you typically use. You can write Python unit tests and execute code from Python REPL, for example:

{% highlight bash %}
$ python
>>> import sample
>>> sample.print_country()
France
{% endhighlight %}

Now, if we want to call it from Vim, some VimL is necessary again. Let's declare a Vim function which will call our
Python function. Add this to the end of `sampleplugin.vim` file:

{% highlight VimL %}
function! PrintCountry()
    python print_country()
endfunction
{% endhighlight %}

Restart test Vim instance, and type: `:call PrintCountry()`. I'm in France, and where are you?

It is not very convenient however to use the `:call` syntax. Typically, Vim plugins provide commands instead, so let's
do just that. Add this line after the function declaration:

{% highlight VimL %}
command! -nargs=0 PrintCountry call PrintCountry()
{% endhighlight %}


Rinse, repeat and type `:PrintCountry` and it still should print the same country. Well done!


## Accessing Vim functionality from Python plugin

Our plugin is quite limited so far: it only spits some text to Vim message area, but doesn't do a lot otherwise. If
we want to do more interesting thins - we need to use `vim` module. It provides Python interface to various Vim
functinality.

For starters, it can simply evalaute expressions writtern in VimL. This is what we previously did to extract a value
of a variable declared in VimL:

{% highlight VimL %}
plugin_root_dir = vim.eval('s:plugin_root_dir')
{% endhighlight %}

`eval` can evalaute any VimL expression and is certinaly not limited to accessing vars. But more often it will be more
convenient to use other `vim` interfaces instead of `eval`.

For example, you can access and modify text in current buffer like so:

{% highlight VimL %}
vim.current.buffer.append('I was added by a Python plugin!')
{% endhighlight %}

As an example, let's implement another command, `InsertCountry`, which would insert the name of the country you are in
at current cursor position. Here is the Python code to add:

{% highlight python %}
def insert_country():
    row, col = vim.current.window.cursor
    current_line = vim.current.buffer[row-1]
    new_line = current_line[:col] + _get_country() + current_line[col:]
    vim.current.buffer[row-1] = new_line
{% endhighlight %}

And, same way as before, let's add according function and command to VimL wrapper script:

{% highlight VimL %}
function! InsertCountry()
    python3 insert_country()
endfunction
command! -nargs=0 InsertCountry call InsertCountry()
{% endhighlight %}

Try it out in a new Vim instance. Position a cursor somewhere in a buffer and run `:InsertCountry`. You can now even
map a key combination for this. For example, run

{% highlight VimL %}
:map <Leader>c :InsertCountry<CR>
{% endhighlight %}

and press `<Leader> c` to run the command! Hey, our plugin just got a major upgrade! Our users can add the mapping to
`~/.vimrc` and their country name is just two key presses away!

Vim plugin can do a lot more interesing things. What is possible and how to use `vim` module is well documented in Vim
itself. Check out help: `:help python-vim` - this is why I mentioned curiosity as a prerequisite previously.


## Configuration

Now this is simple. We already saw everything we need to provide a configuration for our plugin. Typically, users will
configure the plugin in `~/.vimrc` file and set some global variables, which we will later access in a plugin and use
to adjust its behaviour. Say, we want to configure our plugin to provide either country names, or ISO codes. Add
following to your `~/.vimrc`:

{% highlight VimL %}
let g:SamplePluginUseCountryCodes = 1
{% endhighlight %}

And then, access it in Python code:

{% highlight python %}
vim.eval('g:SamplePluginUseCountryCodes')
{% endhighlight %}

Heads up! `eval` will only return a string, list or a dict, depending on type of data used in VimL. In this case, it
is a string, so normally you would actually use it like so:

{% highlight python %}
use_codes = vim.eval('g:SamplePluginUseCountryCodes').strip() != '0'
{% endhighlight %}

You can technically ask users to use 'true' and 'false' in this case for example, but it is good idea to stick to the
behaviour users are already used to with the majority of other plugins, which is using 0 and 1 for this.


## Getting a bit more sophisticated

We are almost done. Let's just finalize our VimL wrapper. It makes sense to add two more features to it:

 - ensure that our plugin is only started when Python is actually available in Vim (this prevents Vim from spitting too
   many errors to the user when Python is not available)
 - and ensure that plugin is initialized once and only once.

Following does precisely that:

{% highlight VimL %}
if !has("python3")
    echo "vim has to be compiled with +python3 to run this"
    finish
endif

if exists('g:sample_plugin_loaded')
    finish
endif

; the rest of plugin VimL code goes here

let g:sample_plugin_loaded = 1
{% endhighlight %}

Now, for example, if our user does something like `:source ~/.vimrc`, we are sure our plugin won't try to run
the initialization code again: won't change `sys.path` again, won't import python modules or execute mode-level code.
Having said that, you don't have any code at module level except for declarations, do you?


## That's it!

Check out `:help python` which contanins a lot of important details.

If you know of other important tricks, or have a good advice - please, leave a comment below. I'm very interested in
further improvments of my Vim plugin development workflow and implementation.

Hope it was useful. Have fun with Vim!
