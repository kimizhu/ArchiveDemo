# Why Can't I Create The WScript Object?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2003 2:25:00 PM

-----

 

Every now and then someone will ask me why the WSH shell object and the WSH network object are creatable from Visual Basic, but the actual root WScript object is not.

 

 

I am always completely mystified by why people ask this\!  Why would you WANT to create the WScript object in a VB app?  What would it do for you?  It isn't creatable as an in-process object because it represents the state of a WSH process, and a VB app is not a WSH process\!  The whole point of the WScript object is that it represents everything about the presently running Windows Script Host.  You can get from it all kinds of properties: the name and version of the script host, the path of the running script, the arguments which were passed in, the standard streams of the process, and whether the script was set to time out or not.  **None of these properties make the slightest bit of sense to access outside of a running instance of WSH\!**  What the heck could this possibly mean?

 

 

Set WScript = CreateObject("WSH.WScript")

Timeout = WScript.TimeOut

 

 

The timeout of what?  You've just got some object there, not a running instance of the script host.  

 

 

What about the methods on the object?  Again, all of them are deeply tied into the underlying script hosting mechanisms.  WScript.CreateObject, GetObject, ConnectObject and DisconnectObject manipulate WSH-owned event sinks on the script dispatch.  I discussed yesterday how deeply tied WScript.Sleep is to the underlying hosting mechanism, and WScript.Echo is not particularly interesting -- there are better ways to create a message box in VB already.

 

 

I just don't get it.  Why do people keep asking me to make the WScript object creatable?  I can't figure out what they think they're going to do with it once they've created it and no one ever gives me a straight answer when I ask**. **It's like asking for the ability to create the Response object without starting ASP\!  What could it possibly mean to have a Response object but no web server process?  **It doesn't make any sense, so we don't let you do it.**

 

 

We *do* provide an object model whereby you can spawn off WSH processes (both locally and remotely) and query their status as they run.  Perhaps that is what these people are looking for?  Remote WSH is a complicated topic in of itself, so perhaps I'll talk about that in a later blog entry.

