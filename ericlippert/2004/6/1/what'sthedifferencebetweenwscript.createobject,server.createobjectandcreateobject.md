# What's the Difference between WScript.CreateObject, Server.CreateObject and CreateObject?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/1/2004 11:49:00 AM

-----

A reader asked me last week  

> I have always used Server.CreateObject in ASP scripts and WScript.CreateObject in WSH scripts, because I was unable to find any \*convincing\* information on the subject and therefore assumed that they were there to be used, and were in some way beneficial... but perhaps they're not?\!

 What exactly is the reasoning behind having additional CreateObjectmethods in ASP and WSH when VBScript already has CreateObject and JScript already has new ActiveXObject ?  (new ActiveXObject is for all intents and purposes identical to CreateObject, so I'll just talk about VBScript's CreateObject for the rest of this article.)  There are two reasons: 

 1)     VBScript and JScript aren't the only game in town.  What if there is a third party language which (inexplicably) lacks the ability to create ActiveX objects?  It would be nice to be able to create objects in such a language, thus the hosts expose said ability on their object models.  (Similarly, IE, WSH and ASP all have "static" object tags as well.)   Of course, this is a pretty silly reason.  First off, no such language comes to mind, and second, why stop at object creation?  A language might not have a method that concatenates strings, but that doesn't mean that we should add one to WSH just because it comes in handy.  We need a better justification. 

 2)     The ASP and WSH versions of CreateObject are subtly different than the VBScript/JScript versions because they have [more information about the host than the language engine posesses](http://blogs.msdn.com/ericlippert/archive/2003/10/08/53175.aspx).  The differences are as follows: 

 WScript.CreateObject with one argument is pretty much identical to CreateObject with one argument.  But with two arguments, they're completely different.  The second argument to CreateObject creates an object on a remote server via DCOM.  The second argument to WScript.CreateObject creates a local object and hooks up its event handlers.  

 Sub Bar\_Frobbed()  
  WScript.Echo "Help, I've been frobbed\!"  
End Sub  
Set foo = CreateObject("Renuberator", "Accounting")  
Set bar = WScript.CreateObject("Frobnicator", "Bar\_")  This creates a renuberator on the accounting server, a frobnicator on the local machine, and hooks up the frobnicator's events to functions that begin with the particular prefix. 

 Remember, in the script engine model the control over how event binding works is controlled by the *host*, not by the *language*.  Both IE and WSH have quite goofy event binding mechanisms for dynamically hooked up events -- IE uses a [single-cast delegate model](http://blogs.msdn.com/ericlippert/archive/2003/12/12/53454.aspx), WSH uses this weird thing with the string prefixes.  

 Server.CreateObject creates the object in a particular *context*, which is important when you are creating *transactable* objects.  Windows Script Components, for example, are transactable objects.  This means that, among other things, you can access the server object model directly from a WSC created with Server.CreateObject , because the object obtains the server context when it is created.  Ordinary CreateObject has no idea what the current page context is, so it is unable to create transactable objects. There are many interesting facets of transactable objects which I know very little about, such as how statefulness and transactability interact, how the object lifetime works and so on. Find someone who writes a blog on advanced ASP techniques and ask them how it works, because I sure don't know. 

 I'll answer the question "what's the difference between WScript.CreateObject and WScript.ConnectObject?" in a later blog entry. 

 I answered the question "What's the difference between GetObject and CreateObject ?" in [this](http://blogs.msdn.com/ericlippert/archive/2004/01/14/58700.aspx "http://weblogs.asp.net/ericlippert/archive/2004/01/14/58700.aspx") blog entry.

