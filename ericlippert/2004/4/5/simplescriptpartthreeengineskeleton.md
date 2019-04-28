# SimpleScript Part Three: Engine Skeleton

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2004 5:32:00 PM

-----

 

Before I get into today's topic I want to quickly address a minor defect that Raymond Chen (who apparently spends his holiday in [Sweden](http://www.plif.com/archive/wc146.gif "http://www.plif.com/archive/wc146.gif") doing remote code-reviews, oddly enough) was good enough to point out to me.  **It is legal to unload a DLL if the only objects that depend on that DLL which are still outstanding are class factories**.  **Therefore, creating the class factory shouldn't add a reference to the DLL**.  If the consumer of the class factory wants to explicitly lock the DLL in memory, they can use the locking mechanism.  This code is wrong: 

ClassFactory::ClassFactory()  
{  
    m\_cref = 1;  
    DLLAddRef();  
} 

Why is that the case?  Why not just use the convention that class factory instances lock the DLL, and if you want the DLL locked, you keep the class factory alive?  Because, though that logic makes perfect sense if the class factory is constructing in-process objects, it does not work if the factory is for a local (out of process) server.  How's that?  Because a local server creates class factories immediately when it starts up and destroys them only when it shuts down.  If creating a class factory locked the local server in memory, then there's an obvious circular reference; the local server would never be shut down.  Thus, the convention for all class factories is that they never lock the code in memory unless asked to, whether they're in-proc or not. 

I've fixed the defect.  Thanks again, Raymond. 

I've uploaded two new files, [engine.cpp](/ericlippert/articles/108025.aspx "http://blogs.msdn.com/ericlippert/articles/108025.aspx") and [engine.h](/ericlippert/articles/108026.aspx "http://blogs.msdn.com/ericlippert/articles/108026.aspx") (and hooked it up to the class factory).  Right now the engine code just knows how to create and destroy itself; no interfaces other than IUnknown are implemented.  As you can see, over the next few entries I'm going to flesh out IActiveScript, IActiveScriptParse, IActiveScriptParseProcedure2 and IObjectSafety.  I already covered the purpose of IObjectSafety in detail in my series "Script and IE Security", which you can read in full by going to my [security archive](/ericlippert/category/2574.aspx?Show=All "http://blogs.msdn.com/ericlippert/category/2574.aspx?Show=All").  [Part Two](/ericlippert/archive/2004/01/13/58403.aspx "http://blogs.msdn.com/ericlippert/archive/2004/01/13/58403.aspx") has a description of the interface.  Why are there three script interfaces instead of one, and what's with that "2"?  

When the *original* script team -- this is before my time\! -- designed the interfaces they realized that there were some things that were going to be common to every possible engine.  Every engine would need a way to communicate back to the host (SetScriptSite), start and stop the engine (SetScriptState) and perform various threading tasks (GetScriptThreadId, etc.)  But how do the script engines do compilation-model dependent things such as compiling up new source code?  Different engines might expose different interfaces to do these kinds of things, so the interface designers left one "base" interface, and if the host wants to use a particular compilation feature, the host and the engine can negotiate via QueryInterface.  

Thus, IActiveScriptParse exposes two methods which expose compilation model features.  ParseScriptText takes in raw source code, parses it, and adds the resulting code to the internal state of the script engine.  If the code contains "global code" it might run, and if it contains procedures then they become available.  This is what is called when you stick a chunk of script inside a plain vanilla \<script\> tag in Internet Explorer.  

ParseScriptlet is very similar, but different -- it takes in source code for the **body of an** **event handler procedure** and adds a new event handler to the internal state.  This is what happens when you have a \<script for="…  block. 

ParseProcedureText, on the IActiveScriptParseProcedure2 interface is similar to ParseScriptlet in that it takes the body of a procedure.  It not only adds the function to the global state, it returns an object which, when invoked, calls the function.  In .NET-speak, it returns a **delegate**. This is handy in the IE event binding model where you bind events to an object by assigning a delegate to a property on the object. 

The reason for the "2" is perfectly straightforward.  When we first implemented IActiveScriptParseProcedure, we left the "name" argument off of the method.  In a later release, we got a feature request to allow the host to control the name of the delegate, so we created a new interface and called it "2".  That's all straightforward.  But if you look in the activscp header file, you'll see IActiveScriptParseProcedure, IActiveScriptParseProcedure2 and IActiveScriptParseProcedure**Old**.  What's up with that, and which ones should we implement? 

The reason why we have IActiveScriptParseProcedureOld was going to be yet another entry in the [Aargh\!](/ericlippert/archive/2004/03/10/87384.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx") series, but I might as well just tell the tale of woe now. 

A good design rule in COM is "*never make an important decision based on the **absence** of an interface*."  Why?  Because if you do, then yo**u are risking that your program breaks if someone implements that interface in the future.**  I learned this rule the hard way because very early versions of Internet Explorer made exactly that mistake. 

In the early days, VBScript did not support the "delegate" model of event binding at all; VBScript uses what we call the "implicit" event binding mechanism whereby we scan the VBScript code looking for procedures with names that match the pattern "objectname underbar eventname".  When we find them, we construct the appropriate hookup code internally and everything is just fine.  

However, doing that search has a **potentially large performance cost**.  If the host decides that it doesn't need the implicit event binding model, the script engine exposes a way to prevent it from happening.  We'll discuss the details later, but for now, I'll just say that if the host wants the feature they "connect" the engine and if they don't, they "start" the engine. 

Also back in the early days, JScript did not support the "implicit" model.  The IE developers therefore implemented the following (bad\!) logic: 

**If the engine *does not* support "delegate" event binding, then "connect" the engine, otherwise "start" the engine. **

That was the logic that shipped with IE4.  One day the IE5 team came to me with a feature request -- they wanted to add "delegate" style event binding to VBScript in IE5.  Piece of cake -- it was actually very easy to implement it in VBScript.  A few lines of code, add the IActiveScriptParseProcedure2 interface implementation, fire it up, and… uh oh.  **Implicit event binding suddenly stopped working in VBScript**.  Aargh\!  It was drivin' me nuts\!  By making a decision to turn a feature **off** based on the **absence** of an interface, IE ensured that we could not implement that interface, ever\! 

The first thing that came to mind was "let's fix that in IE5", which, of course, we did -- IE5 used a much more sensible algorithm to determine whether to connect the engine or not.  Problem solved?  No.  That wasn't good enough, because at the time it was a supported, legal, by-design scenario to install new script engines without installing a new browser.  That meant that if an IE4 customer got the new VBScript engine, their implicit event handlers would stop working\! 

What we needed to do was to retire the old interfaces and ensure that no one would ever use them again.  *Normally* we'd create IActiveScriptParseProcedure3, but I wanted to ensure that no one accidentally use the old interface, so I renamed IActiveScriptParseProcedure2 to IActiveScriptParseProcedureOld, and created a new, different IActiveScriptParseProcedure2.  Only JScript implements IActiveScriptParseProcedure and IActiveScriptParseProcedureOld.  If you implement those interfaces then IE3 and IE4 will not connect your engine.  But given that IE3 and IE4 are a pretty small percentage of the browser market these days, we will dispense with them entirely and only use the new IActiveScriptParseProcedure2.  

I've seen this same mistake in persistence code. (In fact, I think I may have made this same mistake myself in the VSTO2 persistence code.  I need to double check that...)  It's very easy to say “If this object doesn't support such and such an interface, reconstitute it as follows“ and then have the reconstitution code broken when a later version of the object adds that interface.

Coming up: soon I'll start to flesh out the engine a little more, and add some multi-threading support.  I'll also discuss the theory and practice of the "state machine" style of programming and how the host initializes the engine.

