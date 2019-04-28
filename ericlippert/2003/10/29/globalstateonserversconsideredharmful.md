# Global State On Servers Considered Harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/29/2003 2:25:00 PM

-----

The other day I noted that extending the built-in objects in JScript .NET is no longer legal in "fast mode".  Of course, this is still legal in "compatibility mode" if you need it, but why did we take it out of fast mode?

 

 

As several readers have pointed out, this is actually a kind of compelling feature.  It's nice to be able to add new methods to prototypes:

 

 

String.prototype.frobnicate = function(){/\* whatever \*/}

var s1 = "hello";

var s2 = s1.frobnicate();

 

 

It would be nice to extend the Math object, or change the implementation of toLocaleString on Date objects, or whatever.

 

 

Unfortunately, it also breaks ASP.NET, which is the prime reason we developed fast mode in the first place.  Ironically, it is not the additional compiler optimizations that a static object model enables which motivated this change\!  Rather, it is the compilation model of ASP.NET.

 

 

I discussed earlier how ASP uses the script engines -- ASP translates the marked-up page into a script, which it compiles once and runs every time the page is served up.  ASP.NET's compilation model is similar, but somewhat different.  ASP.NET takes the marked-up page and translates it into a **class** that extends a standard page class.  It compiles the derived class once, and then every time the page is served up **it creates a new instance of the class** and calls the Render method on the class.   

 

 

So what's the difference?  The difference is that multiple instances of multiple page classes may be running in the same application domain.  In the ASP Classic model, each script engine is an entirely independent entity.  In the ASP.NET model, page classes in the same application may run in the same domain, and hence can affect each other.  We don't want them to affect each other though -- the information served up by one page should not depend on stuff being served up at the same time by other pages.

 

 

Now I'm sure you see where this is going.  Those built-in objects are shared by all instances of all JScript objects in the same application domain.  Imagine the chaos if you had a page that said:

 

 

String.prototype.username = FetchUserName();

String.prototype.appendUserName = function() { return this + this.username; };

var greeting = "hello";

Response.Write(greeting.appendUserName());

 

 

Oh dear me.  We've set up a race condition.  Multiple instances of the page class running on multiple threads in the same appdomain might all try to change the prototype object at the same time, and the last one is going to win.  Suddenly you've got pages that serve up the wrong data\!  That data might be highly sensitive, or the race condition may introduce logical errors in the script processing -- errors which will be nigh-impossible to reproduce and debug.

 

 

A global writable object model in a multi-threaded appdomain where class instances should not interact is a recipe for disaster, so we made the global object model read-only in this scenario.  If you need the convenience of a writable object model, there is always compatibility mode.

