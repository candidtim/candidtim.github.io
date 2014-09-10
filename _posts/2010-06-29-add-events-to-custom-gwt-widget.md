---
layout: post
title:  "Adding events to a custom GWT widget"
date:   2010-06-29
categories: gwt
---
This post is a "part 2" of the
[Full-fledged GWT Rich Text Editor widget]({% post_url 2010-06-25-gwt-rich-text-editor %}). It is not necessary though
that you read previous one, to dive into here. In previous post I described how to create your own TextEditor GWT
widget by wrapping the TinyMCE Java Script library. In this article I will continue to enhance the widget and will add
a support for a custom event - "save" event in particular.

# Design the event handling from user standpoint

What the widget lacks is an ability to save a document being edited. Without this ability its usage is limited to
embedding it to certain form, having its own submit button. In fact, there is no even a "Save" button in a widget
itself.

Hm..., there is one in the TinyMCE, however. Looks we are lucky - we might merely add one to the widget - it is a
matter of one line of code, as the widget is flexible enough and configurable. But we still won't be able to hook the
"save" event, unless we provide a callback Java Script function. But our idea with the widget was - none of the JSNI
code for widget users. So, we will hide it all inside. Here comes its usage concept:

{% highlight java %}
TextEditor editor = new TextEditor();
// configure editor as needed (size, default content, etc.)
editor.addSaveHandler(new SaveHandler() {
  public void onSave(SaveEvent event) {
    ...
  }
});
{% endhighlight %}

For me - looks nice :) So, let's sum up what we need to create and manage a new custom event:

 1. Create an event class - it may store any information related to the event, its source, attributes, etc.
 2. Create an event handler interface - it will be implemented by widget users and it will be registered within the widget itself
 3. Suite the widget with a method to register an event handler
 4. And, sure enough, implement an event management in the widget

# Add support for GWT events

Good news is that GWT already has almost everything what we need to manage events. GWT not only defines necessary
interfaces and abstract classes, but even provides us with a HandlerManager - this "utility" will track all our event
handlers and will fire events on them when we need it to. Thus, as we stick to GWT event model, we should implement our
event and handler according to GWT standards.

# Event

SaveEvent is an event fired up when user clicks save button. In fact it won't hold much information within itself, and
so its implementation is pretty straightforward. We only need to subclass a GwtEvent, as we want to use GWT's event
management means.

{% highlight java %}
public class SaveEvent extends GwtEvent<SaveEventHandler>
{
  public static Type<SaveEventHandler> TYPE = new Type<SaveEventHandler>();


  @Override
  public Type<SaveEventHandler> getAssociatedType()
  {
    return TYPE;
  }


  @Override
  protected void dispatch(SaveEventHandler handler)
  {
    handler.onSave(this);
  }
}
{% endhighlight %}

Note that we added a TYPE constant and implemented getAssociatedType() method - this is needed as we will use GWT's
HandlerManager and it will "map" events and handlers by their event type. We will also need this constant to be
public - we will get to it in a moment.

Another one point to note is that event also implements a dispacth() method - otherwise how would GWT know the event
handler's method to execute? Instead of trying to guess it - it delegates the event dispatching to us :) It is pretty
simple anyway.

We used an event handler already... Here it is:

# EventHandler

Similarly, as we use GWT's event management means, our SaveEventHandler will extend an EventHandler interface. Our
interface itself is even simpler than SaveEvent - obviously it should be that simple as we want our users to start
using it with minimal efforts:

{% highlight java %}
public interface SaveEventHandler extends EventHandler
{
  void onSave(SaveEvent event);
}
{% endhighlight %}

# Register and track event handlers

Finally, we want to suite our widget with a way to add SaveEventHandlers to it. We agreed already that we will use a
GWT's HandlerManager to manage "save" event. But we don't want our users to access it directly, rather we want them to
use a convenient method, like addSaveHandler(SaveEventHandler).

{% highlight java %}
public class TextEditor extends TextArea implements EntryPoint
{
  ...

  private HandlerManager handlerManager = new HandlerManager(null);

  ...

  public void addSaveHandler(SaveEventHandler handler)
  {
    handlerManager.addHandler(SaveEvent.TYPE, handler);
  }

  ...
}
{% endhighlight %}

# Fire an event

Pretty cool, as you may notice we didn't much work to add an event. Well, it is not fired up anywhere yet, but not a
huge work too. We'll see now.

In fact, TinyMCE already implements an ability to "catch" the user on clicking the "Save" button. So, this means we
only need to add some JSNI code to do so, and link it together with our event model. To do so, we need to initialize
the TinyMCE "save" plug-in, add a save button, and register so-called "save callback" function which is executed by
TinyMCE. Hm, for me - I don't think it is a good idea to implement everything in the widget - in our implementation we
will only ensure there is a "save" plug-in enabled, and add a callback function. We will let widget users to add a
"save" button were they need it (and if they need it).

{% highlight java %}
public class TextEditor extends TextArea implements EntryPoint
{
  ...

  private void initialize()
  {
    // initialize the stuff needed for "save" event
    addPlugin("save"); // ensure it is always there, even if widget user forgot to add it
    // (we also assume here at least "save" button is added - otherwise how would end user click it?)
    addOption("save_onsavecallback", "saveHandler");
    registerSaveHandler();

    ...
  }


  private native void registerSaveHandler()
    /*-{
      var thisTextEditor = this;
      $wnd.saveHandler = $entry(
        function() {
          thisTextEditor.@com.nimble.gwtaddons.texteditor.client.TextEditor::onSave()();
        }
      );
    }-*/;



  private void onSave()
  {
    handlerManager.fireEvent(new SaveEvent());
  }

  ...
}
{% endhighlight %}

# The end

That is all for it. Congratulations, now your widget is equipped with a brand new custom event handler.

Q.E.D.
