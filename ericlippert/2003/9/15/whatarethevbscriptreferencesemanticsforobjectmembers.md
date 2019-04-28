# What are the VBScript reference semantics for object members?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2003 7:36:00 PM

-----

Clearly in order for VBScript to support variable referencing there has to be a variable to reference.  Consider our earlier example:

 

 

Sub Change(ByRef XYZ) 

   XYZ = 5 

End Sub 

Dim ABC 

ABC = 123 

Change ABC 

 

 

If that had been  Change (ABC) then, based on what you know from two posts ago, you’d know that this passes ABC byval, not byref.  So the assignment to XYZ would NOT change ABC in this case.

 

 

Basically, the rule is pretty simple -- if you want to pass a variable by reference, you've got to pass the variable, period.   

 

 

This series of posts was inspired by an intrepid scripter who was trying to combine our previous two examples.  He had a program that looked something like this:

 

 

 

 

Class Foo

   Public Bar

End Class

Sub Change(ByRef XYZ) 

   XYZ = 5 

End Sub 

Dim Blah

Set Blah = New Foo

Blah.Bar = 123

Change Blah.Bar 

 

 

This in fact does not change the value.  This passes the value of Blah.Bar, not a reference to Blah.Bar.

 

 

The scripter asked me "why does this not work the way I expect?"  Here's my Socratic dialog reply to him:

 

 

Q: **Why does this not work the way I expect?** 

 

 

A: Because your expectations are inconsistent with the real universe.  Adjust your expectations and they'll start being met\!

 

 

Q: **That is remarkably unhelpful. Let me rephrase: What underlying design principle did the VBScript developers use to justify this decision to pass by value, not reference?**

 

 

A: The fundamental principle that governs this case was "do not be unnecessarily different from VB6."  VB6 does the same thing.  (Try it if you don't believe me\!)

 

 

Q: **You are begging the question.  Why does VB6 do that?**

 

 

A: Probably for backwards compatibility with VB5, 4, 3, 2 and 1, which incidentally was called "Object Basic".  Ah, the halcyon days of my youth.

 

 

Q: **More question begging\!  What was the initial justification on the day that by-reference calling was added to VB?**

 

 

A: That is lost in the mists of time.  That was like ten years ago, dude\!  There are not very many of the original design team left.  I was an intern at the time and they weren't exactly consulting me on these sorts of decisions on a regular basis.  It wasn't so much "Eric, what do you think about these by reference semantics?" as "Eric, the OLE Automation build machine needs more memory, here's a screwdriver."

 

 

However, you're in luck.  I seem to recall back in the dim mists of time someone telling me something about wanting to avoid copy-in-copy-out semantics on COM objects.  Suppose for example you said:

 

 

Set Frob = CreateObject("BitBucket.Frobnicator")

SetToFive Frob.Rezrov

 

 

OK, so now what happens?  This isn't a VB class, this is some third party COM object.  **COM objects do not have property slots, they have getter/setter accessor functions.** There is no way to pass the value of Frob.Rezrov by reference because VB does not have psychic powers which tell it where in memory the implementers of BitBucket.Frobnicator happened to store the value of the Rezrov property.  

 

 

Given that, how could you implement byref semantics?  You could implement copy-in-copy-out semantics\!  VB would have to create a memory location, fill it with the value returned by get\_Rezrov, pass the address of that location to SetToFive, and then upon SetToFive's return, it would have to call Frob::set\_Rezrov with the new value put into the buffer.  

 

 

Easy, right?  Well, it gets weird once you start thinking about non-trivial functions.  Consider the case where SetToFive does NOT change the value of the by-ref value.  That call to set\_Rezrov may have side effects -- do we really want to call it if nothing changed?  It seems like that could potentially cause badness, and certainly cause poor performance.  In a "realio-trulio byref" system we'd expect zero sets if there was no change but in copy-in-copy-out we end up with one call to the setter regardless.  How could we avoid that unwanted call?  

 

 

Well, we could create yet another temporary storage to keep the original value around and do a comparison when SetToFive returns.  (Note that I've just waved my hands there; I'm assuming that the two values can sensibly be compared.  Comparing two things for equality is non-trivial, but that's another posting.)

 

 

Anyway, what if the temporary storage variable changed during the execution of SetToFive and then changed back?  In that case we'd expect two calls to the setter, but actually end up with no calls\!

 

 

Basically, naïve copy-in-copy-out doesn't provide particularly good fidelity with true byref addressing. The original designers of VB decided that it was simply not worth the trouble to do it at all.   It is much easier to simply say that members of COM objects do not get copy-in-copy-out semantics, and therefore they cannot be passed by reference.  If you're going to make that restriction for some COM objects, it seems perverse to say "we'll do this for third party COM objects but not for VB class objects."  Thus, VBScript does not support passing object properties by reference.

