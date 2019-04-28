<div id="page">

# VBScript Default Property Semantics

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/30/2005 1:15:00 PM

-----

<div id="content">

Here’s a question I recently got about VBScript, where by "recently" I mean August 28th, 2003. This code works just fine: Set myObject = CreateObject("myObject")  
myObject.myName = "Eric"  
WScript.Echo myObject ' myName is the default property, so prints "Eric" But myObject = "Robert" doesn't set the default property, it sets the variable to the string. Why does reading work but writing fail? This works in VB6, why not VBScript?  In a strongly typed language such as VB6 the compiler can look at the compile-time types of the left and right sides of an assignment and determine that the type of the left hand side is an object, the right hand side is a string, and the object has a default property which is a string.  The VB6 compiler can then generate the appropriate code to assign the string to the default property. But what if you wrote a VB6 program using only variants?  When the compiler sees foo = bar typed as variants it cannot tell whether this means "set the default property of foo to bar" or "set foo to bar".  The VB compiler chooses the latter every time. VBScript is a weakly typed subset of VB -- in VBScript, everything is a variant.  So in VBScript, all assignments are treated as though the value is actually being assigned to the variable, not the default property. Therefore this is by design - this is for compatibility with VB6's behaviour.  Had we written the same program in weakly-typed VB6, we'd get the same result.  I hear you exclaiming "The fact that a difference in available type information leads to a difference in run-time behaviour violates a basic principle of programming language design\!  Namely, the principle that late-bound calls have exactly the same semantics as early-bound calls\!"  Indeed, as I've mentioned before, VB6 and VBScript violate this principle in several places.  Default properties were, in my opinion, a bad idea all around, for this and other reasons.  They make the language harder to parse and hence harder to understand.  In particular, *parameterless* default properties make very little sense in VBScript.  However, we are stuck with them now. However, I should call out that there are some things we can do at runtime. Suppose blah is an object with a property fred that is an object, and fred has a default property which is a string. If you say foo = blah.fred then foo *is* assigned the default value of fred even though we lack the compile-time type information.  foo isn't set to the object fred. Of course, in this case we know to fetch the default property at runtime *because we know the runtime type* of fred and also know that there is no Set keyword.  These two facts are sufficient to cause the runtime engine to always fetch the default property. Or suppose you have an object Baz with a property Table, Table has a default *parameterized* property Item which returns an object that has a string property Blah: Then this works just fine: x = Baz.Table(1).Blah But why? In this case we do *not* have enough compile-time information to determine that we really should call Baz.Table.Item(1).Blah.  The script engine has every reason to believe that Table is a function or parameterized property of Baz. This problem is solved by pushing it off to the implementation\! The rule for implementers of IDispatch::Invoke is if all of the following are true:

  - the caller invokes a property
  - the caller passes an argument list
  - the property does not actually take an argument list
  - that property returns an object
  - that object has a default property
  - that default property takes an argument list

then invoke the default property with the argument list. Strange but true. Perhaps unsurprisingly, not very many people know about that rule\!  This is yet another reason to never, ever write your own implementation of IDispatch::Invoke.  It also leads to some mind-numbingly complex code in the VBScript IntelliSense engine that Visual Studio uses.  Making default property IntelliSense work with such a dynamically typed language was a piece of work, lemme tell ya. As I mentioned earlier, there are [in fact some odd corner cases that are slightly different between VB6 and VBScript IntelliSense](http://blogs.msdn.com/ericlippert/archive/2004/05/04/125893.aspx).  I also talked a bit about [left-hand-side default property semantics](http://blogs.msdn.com/ericlippert/archive/2004/05/04/125837.aspx) earlier.

</div>

</div>

