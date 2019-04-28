<div id="page">

# SimpleScript, Part Zero

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/29/2004 10:40:00 AM

-----

<div id="content">

<span>I'm going to embark upon something ambitious here.  I got an email from Mike, our friendly support engineer on the east coast with the subject line "**<span>Need lots of info</span>**".  Apparently he was contacted by a customer who wants to implement a **new script engine** that **supports debugging**, and, here's the part that you've got to really grit your teeth over: **in C\#.**  </span>

<span></span>

<span>My initial response was, of course, *"They're crazy -- talk them out of it\!"*  Not to put too fine a point on it, that's an immense amount of work in any language.  But C\#? The script interfaces were designed long, long before there was managed code.  It's quite likely that it's going to be very hard to write a straightforward interop layer for those interfaces.  "Could they use the managed IVsa interfaces instead?" I asked Mike. </span>

<span></span>

<span>I haven't heard back yet, but I got to thinking last night that maybe it would be interesting to develop a script engine from scratch, describing every interface as I implement it, giving some of the design history behind the interfaces, etc, etc, etc.  I could start it in C++ and then see whether there were ways to make it work well in managed code, etc. </span>

<span></span>

<span>I could then blog a little bit of the code every few days.  This would keep me in blog topics for weeks, certainly.  Probably months.  All of this stuff happens in my spare time, and I have precious little of that, particularly given the number of bugs I have to fix before the Whidbey beta.  But I'll see what I can do. </span>

<span></span>

<span>I'm not sure that a piecemeal daily blog approach is even the right way to handle this kind of material, but I'm willing to make the experiment.  </span>

<span></span>

<span>This is probably how it will go: </span>

<span></span>

<span>Phase One: Basic Infrastructure  
</span><span><span>-<span>       </span></span></span><span>dll entrypoints  
</span><span><span>-<span>       </span></span></span><span>registration  
</span><span><span>-<span>       </span></span></span><span>class factory  
</span><span><span>-<span>       </span></span></span><span>engine skeleton </span>

<span>Phase Two: Building the engine  
</span><span><span>-<span>       </span></span></span><span>script engines as state machines  
</span><span><span>-<span>       </span></span></span><span>threading concerns  
</span><span><span>-<span>       </span></span></span><span>named items  
</span><span><span>-<span>       </span></span></span><span>event handling </span>

<span>Phase Three: Processing the language  
</span><span><span>-<span>       </span></span></span><span>grammar  
</span><span><span>-<span>       </span></span></span><span>lexer  
</span><span><span>-<span>       </span></span></span><span>parser  
</span><span><span>-<span>       </span></span></span><span>code generator  
</span><span><span>-<span>       </span></span></span><span>interpreter  
</span><span><span>-<span>       </span></span></span><span>runtime library </span>

<span>Phase Four: Debugger support  
</span><span><span>-<span>       </span></span></span><span>breakpoints  
</span><span><span>-<span>       </span></span></span><span>evaluation  
</span><span><span>-<span>       </span></span></span><span>syntax colouring  
</span><span><span>-<span>       </span></span></span><span>context </span>

<span>Phase Five: Managed Code  
</span><span><span>-<span>       </span></span></span><span>? </span>

<span></span>

<span>That's just off the top of my head.  We'll probably mix it up a bit as we go. </span>

</div>

</div>

