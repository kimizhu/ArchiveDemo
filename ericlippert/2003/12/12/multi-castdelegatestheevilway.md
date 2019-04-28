# Multi-cast delegates the evil way

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/12/2003 2:00:00 PM

-----

 

 

A lot of people have asked me over the years how various kinds of event binding work.  Basically, event binding works like this: 

1\)     Someone clicks on a button,  
2\)     then a miracle happens, and...  
3\)     the button's event handlers execute. 

It's that second step that people struggle with. 

First, some terminology.  I studied applied mathematics, and somethings we talked about quite a bit were **sources** and **sinks**. Sources produce something -- a faucet produces water at a certain rate, for example.  A sink takes that water away.  We'll borrow this terminology for our discussion of events.  An **event source** is something that **produces** events, like a button or a timer.  An **event sink** is something that **consumes** events, like an event handler function.  (Event sinks are also sometimes called "listeners", which mixes metaphors somewhat, but that's hardly unusual in this profession.) 

This terminology leads to a rather unfortunate homonymy -- when I first heard "this method **sinks** the click event", I heard "this method **syncs** the click event".  When we talk about event sinks, we're talking about the consumer of something, not about synchronizing two things in time.  (Sinks, of course, can be asynchronous...) 

The miracle actually isn't that miraculous.  Implementing event sources and sinks requires two things: first, a way to wrap up a function as an object, such that when the source wants to "fire" the event, all it does is invokes the sink's wrapper.  Second, a way for the thread to detect that the button, or whatever, has been pressed and thereby know to trigger the sink wrappers.  

An explanation of the magic behind the latter would take us fairly far afield.  Suffice to say that in IE, the details of how that mouse press gets translated into windows messages and how those messages are dispatched by the COM message loops behind the scenes are miracles that I don't want to talk about in this article.  I'm more interested in those wrappers. 

In the .NET world, an object that can be invoked to call a function is called a **delegate**.  In JScript Classic, all functions are first-class objects, so in a sense, all functions are delegates.  How does the source know that the developer wishes a particular delegate (ie, event sink) to be invoked when the event is sourced? 

Well, in IE, it's quite straightforward: 

function doSomething() {  }  
button1.onclick = doSomething;  // passes the function object, does not call the function 

But here's an interesting question -- what if you want TWO things to happen when an event fires?  You can't say 

function doSomething() {  }  
function doOtherThing() {  }  
button1.onclick = doSomething;  
button1.onclick = doOtherThing; 

because that will just replace the old sink with the new one.  The DOM only supports "single-cast" delegates, not "multi-cast" delegates.  A given event can have no more than one handler in this model. 

What to do then?  The obvious solution is to simply combine the two. 

function doSomething() {  }  
function doOtherThing() {  }  
function doEverything() { doSomething(); doOtherThing(); }  
button1.onclick = doEverything; 

But what if you want to **dynamically** add new handlers at runtime?  I recently saw an inventive, clever, and incredibly horribly awful solution to this problem.  Some code has been changed to protect the guilty. 

function addDelegate( delegate, statement) {   
  var source = delegate.toString() ;    
  var body = source.substring(source.indexOf('{')+1, source.lastIndexOf('}')) ;   
  return new Function(body + statement);  
} 

Now you can do something like this: 

function dosomething() { /\* whatever \*/ }  
button1.onclick = dosomething;  
// ... later ...  
button1.onclick = addDelegate(button1.onclick, "doOtherThing();") ; 

That will then decompile the current delegate, extract the source code, append the new source code, recompile a new delegate using "eval", and assign the new delegate back. 

OK, people, pop quiz.  You've been reading this blog for a while.  What's wrong with this picture?  Put your ideas in comments and I'll discuss them in my next entry. 

This is a gross abuse of the language, particularly considering that this is so easy to solve in a much more elegant way.  The way to build multi-cast delegates out of single-cast delegates is to -- surprise -- build multi-cast delegates out of single cast delegates.  Not decompile the single-cast delegate, modify the source code in memory, and then recompile it\!  There are lots of ways to do this.  Here's one: 

function blur1(){whatever}  
function blur2(){whatever}  
  
var onBlurMethods = new Array();  
  
function onBlurMultiCast() {   
  for(var i in onBlurMethods)   
    onBlurMethods\[i\]();  
}  
blah.onBlur = onBlurMultiCast;  
onBlurMethods.push(blur1);  
onBlurMethods.push(blur2); 

 

 

I'll talk about VBScript and JScript .NET issues with event binding another time.  

 

I'm on vacation for the next three weeks, so I might have lots of time for blogging, or lots of other things to do.  So if you don't hear from me, I'll be back in the new year.  Have a festive holiday season\!

