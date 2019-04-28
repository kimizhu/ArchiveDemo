<div id="page">

# Use vs. Mention in JScript Doesn't Come For Free

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/20/2004 10:16:00 AM

-----

<div id="content">

Before today's entry, a quick note. Work has just gotten **insanely** busy as we push towards getting VSTO ready for the Whidbey release. I likely won't have much time to blog over the next couple weeks, so the blog refresh rate is going to go down for a while. I have a collection of pre-written articles -- such as this one -- that I'll dip into every now and then when I have a few spare minutes, but I likely won't be very topical or responsive for a bit. OK, onward; a while back a reader asked what the difference was between document.write("hello"); and foo = document.write;  
foo("hello"); from the point of view of what actually happens "behind the scenes" in the IDispatch calls.  The reader noted in particular that though this trick works in IE, it **doesn't** work in WSH: foo = WScript.Echo; // Nope, sorry.  
foo("hello"); Here's the deal.  The IE object model has been specially designed with JScript in mind.  Unlike VBScript, JScript makes a distinction between **naming** a function and **calling** a function.  When you say abc = blah.baz(); that calls the baz function and results in whatever it returns.  But abc = blah.baz; assigns the baz **function object** itself to abc; it essentially makes an **alias**. The IDispatch interface takes flags which determine whether the caller wants to fetch the named property or call the named function. Many object models do not honour those flags though, because no language before JScript really took advantage of the distinction.  In fact, object models designed to be called from VB or VBScript often don't even check the flag -- if it says "fetch the property" or "call the method", well, it just calls the method either way. So here's what happens when you say foo = document.write;  First, JScript attempts to resolve "document".  It can't find a local or global variable called that, so it asks the global window object for the document property.  IE gives back the document object.  JScript then asks the document object to give back the value of the write property.  **IE creates an object which has a default method**.  The default method calls the write function, but no one calls the method yet -- we just have an object which, when invoked, will call the mehtod.  JScript assigns the object to foo.  Then when you call foo("hello"); JScript invokes the default method on the object, which calls the write method. The WSH object model was not designed with this in mind.  It does not make a distinction between naming a function and calling it, so you can't use this trick.  WScript.Echo; does not give back a function object that can be invoked later.

</div>

</div>

