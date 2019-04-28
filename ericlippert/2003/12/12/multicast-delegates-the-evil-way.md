<div id="page">

# Multi-cast delegates the evil way

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/12/2003 2:00:00 PM

-----

<div id="content">

<span> </span>

<span> </span>

<div>

<span>A lot of people have asked me over the years how various kinds of event binding work.  Basically, event binding works like this: </span>

<span></span>

<span>1)</span><span>     </span><span>Someone clicks on a button,  
</span><span>2)</span><span>     </span><span>then a miracle happens, and...  
</span><span>3)</span><span>     </span><span>the button's event handlers execute. </span>

<span></span>

<span>It's that second step that people struggle with. </span>

<span></span>

<span>First, some terminology.  I studied applied mathematics, and somethings we talked about quite a bit were **<span>sources</span>** and **<span>sinks</span>**. Sources produce something -- a faucet produces water at a certain rate, for example.  A sink takes that water away.  We'll borrow this terminology for our discussion of events.  An **<span>event source</span>** is something that **<span>produces</span>** events, like a button or a timer.  An **<span>event sink</span>** is something that **<span>consumes</span>** events, like an event handler function.  (Event sinks are also sometimes called "listeners", which mixes metaphors somewhat, but that's hardly unusual in this profession.) </span>

<span></span>

<span>This terminology leads to a rather unfortunate homonymy -- when I first heard "this method **<span>sinks</span>** the click event", I heard "this method **<span>syncs</span>** the click event".  When we talk about event sinks, we're talking about the consumer of something, not about synchronizing two things in time.  (Sinks, of course, can be asynchronous...) </span>

<span></span>

<span>The miracle actually isn't that miraculous.  Implementing event sources and sinks requires two things: first, a way to wrap up a function as an object, such that when the source wants to "fire" the event, all it does is invokes the sink's wrapper.  Second, a way for the thread to detect that the button, or whatever, has been pressed and thereby know to trigger the sink wrappers.  </span>

<span></span>

<span>An explanation of the magic behind the latter would take us fairly far afield.  Suffice to say that in IE, the details of how that mouse press gets translated into windows messages and how those messages are dispatched by the COM message loops behind the scenes are miracles that I don't want to talk about in this article.  I'm more interested in those wrappers. </span>

<span></span>

<span>In the .NET world, an object that can be invoked to call a function is called a **<span>delegate</span>**.  In JScript Classic, all functions are first-class objects, so in a sense, all functions are delegates.  How does the source know that the developer wishes a particular delegate (ie, event sink) to be invoked when the event is sourced? </span>

<span></span>

<span>Well, in IE, it's quite straightforward: </span>

<span></span>

<span>function doSomething() {  }  
</span><span>button1.onclick = doSomething;  // passes the function object, does not call the function </span>

<span></span>

<span>But here's an interesting question -- what if you want TWO things to happen when an event fires?  You can't say </span>

<span></span>

<span>function doSomething() {  }  
</span><span>function doOtherThing() {  }  
</span><span>button1.onclick = doSomething;  
</span><span>button1.onclick = doOtherThing; </span>

<span></span>

<span>because that will just replace the old sink with the new one.  The DOM only supports "single-cast" delegates, not "multi-cast" delegates.  A given event can have no more than one handler in this model. </span>

<span></span>

<span>What to do then?  The obvious solution is to simply combine the two. </span>

<span></span>

<span>function doSomething() {  }  
</span><span>function doOtherThing() {  }  
</span><span>function doEverything() { doSomething(); doOtherThing(); }  
</span><span>button1.onclick = doEverything; </span>

<span></span>

<span>But what if you want to **<span>dynamically</span>** add new handlers at runtime?  I recently saw an inventive, clever, and incredibly horribly awful solution to this problem.  Some code has been changed to protect the guilty. </span>

<span></span>

<span>function addDelegate( delegate, statement) </span><span>{   
</span><span>  var source = delegate.toString() ;    
</span><span>  var body = source.substring(source.indexOf('{')+1,</span><span> </span><span>source.lastIndexOf('}')) ;   
</span><span>  return new Function(body + statement);  
</span><span>} </span>

<span></span>

<span>Now you can do something like this: </span>

<span></span>

<span>function dosomething() { /\* whatever \*/ }  
</span><span>button1.onclick = dosomething;  
</span><span>// ... later ...  
</span><span>button1.onclick = addDelegate(button1.onclick, "doOtherThing();") ; </span>

<span></span>

<span>That will then decompile the current delegate, extract the source code, append the new source code, recompile a new delegate using "eval", and assign the new delegate back. </span>

<span></span>

<span>OK, people, pop quiz.  You've been reading this blog for a while.  What's wrong with this picture?  Put your ideas in comments and I'll discuss them in my next entry. </span>

<span></span>

<span>This is a gross abuse of the language, particularly considering that this is so easy to solve in a much more elegant way.  The way to build multi-cast delegates out of single-cast delegates is to -- surprise -- build multi-cast delegates out of single cast delegates.  Not decompile the single-cast delegate, modify the source code in memory, and then recompile it\!  There are lots of ways to do this.  Here's one: </span>

<span></span>

<span>function blur1(){whatever}  
</span><span>function blur2(){whatever}  
  
</span><span>var onBlurMethods = new Array();  
  
</span><span>function onBlurMultiCast() </span><span>{   
</span><span>  </span><span>for(var i in onBlurMethods)   
</span><span>  </span><span>  </span><span>onBlurMethods\[i\]();  
}  
</span><span>blah.onBlur = onBlurMultiCast;  
</span><span>onBlurMethods.push(blur1);  
</span><span>onBlurMethods.push(blur2); </span>

<span></span><span> </span>

<span></span><span> </span>

<span>I'll talk about VBScript and JScript .NET issues with event binding another time.  </span>

<span></span><span> </span>

<span>I'm on vacation for the next three weeks, so I might have lots of time for blogging, or lots of other things to do.  So if you don't hear from me, I'll be back in the new year.  Have a festive holiday season\! </span>

<span></span>

</div>

</div>

</div>

