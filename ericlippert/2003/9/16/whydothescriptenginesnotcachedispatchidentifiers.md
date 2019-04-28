# Why do the script engines not cache dispatch identifiers?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 8:44:00 PM

-----

You COM programmers out there are intimately familiar with the IDispatch interface, I'm sure.  To briefly sum up for the rest of you, the point of IDispatch is that it allows a caller to call a function without actually knowing **at compile time** any details about the name or signature of that function.  The caller passes a method name to IDispatch::GetIdsOfNames to get a dispatch identifier -- an integer -- which identifies the method, and then calls IDispatch::Invoke with the dispid and the arguments.  The implementation of Invoke is then responsible for analyzing the arguments and calling the actual function on the object's virtual function table.

 

 

This is how VBScript and JScript work.  When you say

 

 

Set Frob = CreateObject("BitBucket.Froboznicator")

Frob.Gnusto 123, "skidoo" 

 

 

what VBScript actually does is pass "Gnusto" to GetIdsOfNames, and then passes the dispid and arguments to Invoke.  The object then does the work of actually translating that into a real function call.  This is how we get away with not having to write a down-to-the-machine-code compiler for VBScript and JScript.

 

 

One of the Fundamental Rules of OLE Automation is that for any object, any two calls to GetIdsOfNames with the same string must return the same number.  This is so that callers can fetch the dispid once and reuse it, rather than having to fetch it anew every time they try to invoke the dispatch object.

 

 

But VBScript and JScript do not cache the dispid.  If you say

 

 

Set Frob = CreateObject("BitBucket.Froboznicator")

Frob.Gnusto 123, "skidoo" 

Frob.Gnusto 10, "lords a leaping" 

 

 

then the script engine will call GetIdsOfNames twice, and get the same value both times.   

 

 

Surely this is wasteful.  Can't we cache that thing?  I mean, it is a small optimization; that call to Invoke is going to dwarf the expense of the GetIdsOfNames.  It just seems like there ought to be something we could do here.

 

 

Appearances can be deceiving.   

 

 

You might first think that every time we see Frob.Gnusto we can use the dispid we grabbed the first time and reuse it.  For example:

 

 

Frob.Gnusto 123, "skidoo" ' OK, Gnusto is dispid 0x1111.  Add to cache "Frob.Gnusto", 0x1111 

Frob.Gnusto 10, "lords a leaping"  ' Look up "Frob.Gnusto" in cache, aha, it is 0x1111 

 

 

This doesn't work.  Consider this scenario:

 

 

Set Frob = CreateObject("BitBucket.Froboznicator")

Frob.Gnusto 123, "skidoo" 

Set Frob = CreateObject("MushySoft.Gronker")

Frob.Gnusto 10, "lords a leaping"   

 

 

Two objects sharing the same variable but with different types may have the same method name but different dispids.  We would have to invalidate our cache every time the variable was changed, which is a lot of work to save very little time.   

 

 

There are other reasons why this doesn't work.  Consider this JScript code:

 

 

var Frob = new ActiveXObject("BitBucket.Froboznicator");

var Frab = new ActiveXObject("BitBucket.Flouncer");

Frob.Gnusto(123, "skidoo"); // Cache "Frob.Gnusto", 0x1111

with(Frab)

{

      Frob.Gnusto(10, "lords a leaping");

}   

 

 

OK, now is that Frab.Frob.Gnusto(10...) , or Frob.Gnusto(10...)?  Frab might have a method Frob which has a method Gnusto which has a different dispid.

 

 

Similarly, there are problems with multiple scopes where local variables may shadow globals.  It's a huge mess.  The net result is that **we can't cache dispids against variable names**.  The variable names are just too ephemeral.

 

 

What about the object POINTERS?  Every object has a unique 32 bit pointer associated with it, right?  Why not cache the dispids against the pointer-and-method-name pair? 

 

 

Unfortunately, that doesn't work either.  Suppose object Frob is 0x12345000 in memory.

 

 

Frob.Gnusto 123, "skidoo" ' OK, Gnusto is dispid 0x1111.  Add to cache "0x12340000.Gnusto", 0x1111 

Frob.Gnusto 10, "lords a leaping"  ' Look up "0x12340000.Gnusto" in cache, aha, it is 0x1111 

 

 

This may look fine, but again, it isn't.   

 

 

Set Frob = CreateObject("BitBucket.Froboznicator")

Frob.Gnusto 123, "skidoo"

Set Frob = CreateObject("MushySoft.Gronker")

Frob.Gnusto 10, "lords a leaping"   

 

 

What happened on that third line of code there?  We threw away the only reference to the current value of Frob, freeing the pointer.  Then the operating system went and created a new object.  The operating system could be very smart about pointer re-use.  It knows that it has a perfectly good free pointer at 0x12340000, so it might re-use it.  Now our cache needs to be invalidated every time an object is freed\!  We must keep track of when every single object is freed -- basically, we have to write a garbage collector for arbitrary COM objects\!

 

 

**To cache dispids you need to ensure that the lifetime of the object is greater than the lifetime of the cache**.  But object lifetimes are so variable, it is very hard to know when the cache is invalid. We once considered adding dispid caching to those objects where we knew that the objects would live longer than the script -- the "window" object in IE, or the "Response" object in IIS, but rejected the proposal as too much complication for very little performance gain.

