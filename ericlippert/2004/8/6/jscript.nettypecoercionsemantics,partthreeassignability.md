# JScript .NET Type Coercion Semantics, Part Three: Assignability

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/6/2004 10:38:00 AM

-----

Before I begin today's technical topic, a quick link to what promises to be a *terrifying*, I mean, *terribly interesting* blog.   **Mario**, the guy who tests the code that I write and thereby keeps me honest, the guy who's application for a backyard barbecue was turned down by the Redmond fire department, the guy who, when I showed him how to use MSBUILD to change his security policy responded with "wonderful\! If I was a woman I would kiss you... but because the lack of some features a strong handshake is more appropiate for both of us,“ -- yes, [that guy has started blogging.](http://blogs.msdn.com/windrago/) 

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Today I'll give a more precise definition of how JScript .NET determines whether one type is assignable to another. Remember, **assignable** is weaker than **promotable** -- promotable means that we are **guaranteed** that every instance of then source type is coercible to the target type.  With assignable, we care only if **some instance** of the source type is coercible to the target type.  Next time, we'll finish up this **incredibly boring series** with the actual coercibility algorithm.  Then I think I'll take a short break and talk about some lighter topics before tackling overload resolution. 

Again, suppose we have an assignment of an expression to a typed field. 

Left Hand Side Expression = Right Hand Side Expression; 

where the compiler can infer the type of the left hand side as LHT and the right hand side as RHT.  I'm going to be a little bit loose in my definition of assignable below in that sometimes I care about the **type** of the right hand expression, but sometimes we have more information than just the type and we want to determine whether the **expression result** is assignable.  It should be clear from the context which way I mean it. 

Unfortunately, the current release of JScript .NET has a number of bugs -- it appears that the compiler does not actually enforce some of these rules at compile time when it could.  Assigning bad constant strings to enumerated types, for example, isn't caught until runtime.  Neither is the rule for double-\>single conversions.  Weird.  I'll mention those to the JScript .NET dev next time I see him.  

We start with three broad categories: 

  - RH Expression is a **compile-time constant. **
  - ****RH Expression is an **array literal. **
  - ****RH Expression is **something else. **

**RHExp is a compile-time constant **

 This is clearly the simplest case.  We know not only the type but the value, so the question of whether RHT is assignable to LHT or not is irrelevant.  All we care about is whether this specific expression is assignable. 

  - If constant RHExp is the **name of a class** then RHExp is assignable to LHT **if and only if** LHT is **System.Type** or **System.Object**. 
  - If LHT is an **enumerated type** and RHExp is a **string** then RHExp is assignable to LHT **if and only if **the string names a member of that enumerated type. 
  - If the constant RHExp is coercible to type LHT then it is assignable.  (More on this next time.) 
  - If RHExp is a **numeric literal** and RHT is **8-byte float** and LHT is a **4-byte float** then RHExp is assignable to LHT. Yes, this is potentially lossy, but there's not much else we can do, so be careful\!  However… 
  - If RHExp is a **compile-time constant 8-byte float** but NOT a numeric literal and LHT is a **4-byte float** then RHExp is assignable to LHT if and only if the string representation of the double and the string representation of that double downcast to single are identical.  (Oddly enough, this rule doesn't seem to be enforced by the compiler.  Looks like a bug.) 

**RHExp is an array literal **

 Array literals are weird because they are treated as JScript arrays or system arrays depending on the context.  It matters very little what specifically RHExp is to determine assignment compatibility: 

  - If LHT is **System.Object** or **System.Array** or **JScript array** then RHExp is assignable to LHT 
  - If LHT is **not some kind of array** then RHExp is **not** assignable to LHT 
  - If LHT is **an array type of known rank and that rank is not 1** then RHExp is **not** assignable to LHT 
  - If LHT is **an array of known type ElemType** then RHExp is assignable to LHType **if and only if every element in the array literal is assignable to ElemType.  **

**RHExp is something else.** 

  - If LHT is **System.Object** then RHT is assignable to LHT.
  - If LHT **any primitive numeric type** and RHT is 8-byte float then RHT is assignable to LHT.
  - If RHT is **promotable** to LHT then obviously RHT is assignable to LHT.
  - If LHT is a **delegate** and RHExp is a **signature-compatible script function** then RHExp is assignable to LHT.
  - If LHT is **not any kind of array** and RHT is **JScript array** then RHT is **not** assignable to LHT.
  - If LHT is **System.Array** and RHT is **JScript array** then RHT is assignable to LHT.  (I believe this gives a warning though.)
  - If LHT is an **array type known to be rank 1** and  RHT is **JScript array** then RHT is assignable to LHT.
  - If LHT is an **array type known to be rank other than 1** and  RHT is **JScript array** then RHT is not assignable to LHT.
  - If LHT is **string** then RHT is assignable -- just about everything can be converted to string.
  - If LHT is **Boolean or any primitive numeric type** and RHT is **string** then RHT is assignable to LHT. 
  - If LHT is **char** and RHT is **string** then RHT is assignable to LHT. 
  - If LHT is **promotable** to RHT then RHT is assignable to LHT. For instance, base classes are assignable to derived classes even though they are not promotable.
  - If LHT and RHT are both **primitive numeric types** then RHT is assignable to LHT.

Those last four may give a compile-time warning, as they're pretty dodgy.

