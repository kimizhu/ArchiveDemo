# A Definite Assignment Anomaly

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/18/2010 6:28:00 AM

-----

**UPDATE: I have discovered that this issue is considerably weirder than the initial bug report led me to believe. I've rewritten the examples in this article; the previous ones did not actually demonstrate the bug.**

 Consider the following code:

 

struct S  
{  
  private string blah;  
  public S(string blah)  
  {  
      this.blah = blah;  
  }  
  public void Frob()  
  { // whatever  
  }  
}

This method body code fragment is legal (though probably ill-advised):

 

S s1 = new S();  
s1.Frob();

Every struct has a default constructor which initializes its fields to their default values; this frobs the zero handle.

What about this?

S s2;  
s2.Frob();

Looks like a definite assignment error, doesn’t it?

An interesting and little-known fact about the C\# compiler is that this is reported as a definite assignment error if the struct is compiled from **source code**, but not if the struct is in a referenced assembly\! What is the reasoning behind this seemingly anomalous behaviour?

Well, consider the following related situation:

struct SS  
{  
  public int x;  
  public int y;  
  public void Bar() {…}  
} 

Here we have the pure unadulterated evil that is a mutable value type. Is this legal?

 

SS ss;  
ss.x = 123;  
ss.y = 456;  
ss.Bar();

Yes, that is legal. The local variable s essentially has two sub-variables, x and y. If x and y are both definitely assigned, then s is also considered to be definitely assigned. **When checking definite assignment on a struct, the compiler actually checks to see whether we know that every *field* of the struct has been initialized.** A constructor call is assumed to automatically initialize every field, but if there is not a constructor call, then we simply track every field independently and then pool the results. (And if the fields are themselves of value type, we do so recursively of course.)

The anomaly happens because when the compiler sees that the type in question is in source code, it tracks the definite assignment of all the fields, detects that “s1.blah” is an unassigned field, and gives a definite assignment error on s1. But when the type in question is from a referenced assembly, **the compiler ignores inaccessible fields of reference type**. We see that there were zero fields initialized out of the zero accessible fields that needed to be initialized, and declare that s1 must be fully assigned\!

**UPDATE: Initially I thought this behaviour was odd and undesirable but plausibly justifiable. However I have since learned that in fact, for reasons still obscure to me, the compiler only ignores inaccessible fields of *reference* type. Inaccessible fields of *value* type are still tracked for definite assignment. This weird inconsistency points strongly in the direction of this being a bug, not a feature. Continuing with my original text...**

This behaviour is undoubtedly *weird*; whether it is *desirable* or not is a judgment call. There certainly is an argument to be made that the user has no ability to do anything about a private field on an imported type; a type they almost certainly did not author themselves, do not know the details of, and cannot change. Whereas if the type is in source code, then the user running the compiler has personal knowledge of its internals. So, the argument goes, we should give the user a pass on having to prove that the fields they cannot access and know nothing about are assigned; just let them be assigned to their default values and don’t worry about them.

There is also the (more compelling to me) argument that we ought to be consistent here and either consider *all* *the fields* all the time, or consider *only the accessible fields* all the time, rather than swapping back and forth between the two choices depending on the details of the compilation process. My choice would be the former; force the user to call the default constructor or use the “default(T)” operator if they really want the all-zero default version of the struct. That makes the intention very clear in the code.

Unfortunately, though I think I’ve argued that this behaviour is plausibly undesirable, we’re stuck with it. There are numerous BCL structs which have no public fields, and *lots* of existing code which uses them without calling the default constructor. Changing now to make this an error is a breaking change with no compelling benefit, and we’re not going to do that. Particularly since the anomaly is essentially harmless; whether we force you to say “s1 = new S()” or not, the handle field is in practice always initialized to its default value.

Nor do I think we should change this by making the code legal in all cases, even if the type is in source code; that’s making a bad situation worse merely in order to achieve consistency. This seems like the very definition of “foolish consistency”.

