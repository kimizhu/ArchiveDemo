# VBScript and JScript Don't Mix, at least in ASP

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/19/2004 10:48:00 AM

-----

A reader wrote me recently to describe a problem that I used to hear fairly often: 

> I write ASP code using VBScript and a coworker of mine uses JScript.  We were wondering if we could combine our code in ASP.  However, it does not seem to be working right.  Here is an example of where we are having trouble: 
> 
> \<SCRIPT LANGUAGE="JScript" RUNAT="Server"\>  
> Response.Write("\<H1\>header\</H1\>");  
> \</SCRIPT\>  
> \<SCRIPT LANGUAGE ="VBScript" RUNAT="Server"\>  
> Response.Write "\<H1\>body\</H1\>"  
> \</SCRIPT\>  
> \<SCRIPT LANGUAGE ="JScript" RUNAT="Server"\>  
> Response.Write("\<H1\>footer\</H1\>");  
> \</SCRIPT\>
> 
> As you can see this code should write: 
> 
> header  
> body  
> footer
> 
> but the ASP engine does not render it like that.

My advice: don't mix languages on one ASP page because, as my correspondent discovered, it doesn't work the way you think it should.  The rendering comes all out of order.  Why? 

Before you read on, if you don't understand the ASP compilation model you should [read my earlier post](/ericlippert/archive/2003/09/18/53046.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/18/53046.aspx") on the subject.  To briefly recap, ASP translates the page into a script, sticks the script into an engine, and then clones the engine a bunch of times if it needs to.  Every time the page is served, the engine is pulled out of the engine pool, the script is run, and everything just works. 

So what happens when you have two languages on the page?  You need two engines, but the mechanism is the same.  The page is compiled up into two scripts, and **ASP runs one engine and then the other.** 

Now it should become clear why this appears to run out of order.  From ASP's perspective, the page above turns into two scripts: 

// Jscript  
Response.Write("\<H1\>header\</H1\>");  
Response.Write("\<H1\>footer\</H1\>"); 

' VBScript  
Response.Write "\<H1\>body\</H1\>" 

The ASP engine does not maintain any information about which **chunk** should execute before another.  It gathers up all the script chunks for each language, compiles them, and then **runs each engine in turn**.  Not only does it lead to seemingly out-of-order execution as shown here, but it also restricts the language blocks from calling each other's methods (because the methods in the not-yet-executed block may not have been initialized yet.)  A detailed discussion of how IE solves this problem deserves another blog entry on its own. 

Mixing languages can also lead to performance problems, because it means that each page is now consuming TWO engines from the engine cache, which means more cache misses, larger working set, etc. 

While I'm on the subject, note that it is in general a bad idea to put a whole lot of "inline" code into \<SCRIPT\> blocks like that.  Ideally you want the server side \<SCRIPT\> blocks to contain only global function definitions, and the \<% %\> blocks to contain only "inline" code.  ASP does not enforce that constraint, but ASP.NET does.  (The reasoning behind that is long enough to require another post.) 

If you really want to have multiple languages on one ASP page, here is a way you can make this work though if you really need to.  You could write a bunch of Windows Script Component objects in different languages, which expose methods.  You could then write an ASP page in one language that calls those methods in the right order.  WSCs are good in ASP because (a) they efficiently cache engine state, just like ASP, and (b) they can use the ASP object model directly if you write them correctly.

