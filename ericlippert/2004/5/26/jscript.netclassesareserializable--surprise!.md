# JScript .NET Classes Are Serializable -- Surprise\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/26/2004 5:29:00 PM

-----

You learn something new every day in this job.  Or, more accurately, some days you learn things again that you'd forgotten years ago.   Someone just asked me why it is that all JScript .NET classes are serializable.  I admit it, my first reaction was "They are?"  But I checked the sources, wrote some code and ran it through ILDASM and sure enough, all JScript .NET classes ARE serializable\!  

 Though I did not write that code, surely I must have known that at *some* point.  Now that I think about it a bit more, it's coming back to me.  I think the idea was that the designers anticipated that there would often be situations in which JScript.NET objects would have to be passed across appdomains.  (Exceptions for instance -- in JScript, you can throw anything.) That means either making classes marshal-by-ref or serializable, and apparently we picked serializable.  

 We could of course have gone the other way, and made developers figure out which classes needed to be serialized, and defaulted to “not serializable“.  But one of the major JScript .NET design principles is "just get it done, already\!"  C\# was designed so that most of the defaults limit behaviour -- things are private until made public, and so on.  In JScript .NET, as much as possible is left open; if you want to tighten it up, you go right ahead.  This is in keeping with the scripty nature of the language, while still recognizing that people do write large programs in script languages. 

 If for some reason you wish a class to be not serializable then you can do this: 

     public System.NotSerialized class MyClass { 

 (Notice that in JScript .NET, you don't need to put ugly brackets around attributes.  The parser is smart enough to figure out that there is an attribute before the class declaration.  However, this significantly complicates the parser; I see why C\# and VB.NET require their delimeters.) 

Offhand I don't see anywhere on MSDN where this feature is documented, though it is possible that my google-fu has failed me.

