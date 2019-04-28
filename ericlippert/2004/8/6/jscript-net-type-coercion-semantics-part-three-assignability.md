<div id="page">

# JScript .NET Type Coercion Semantics, Part Three: Assignability

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/6/2004 10:38:00 AM

-----

<div id="content">

<span>Before I begin today's technical topic, a quick link to what promises to be a *terrifying*, I mean, *terribly interesting* blog.  </span> <span>**Mario**, the guy who tests the code that I write and thereby keeps me honest, the guy who's application for a backyard barbecue was turned down by the Redmond fire department, the guy who, when I showed him how to use MSBUILD to change his security policy responded with "wonderful\! If I was a woman I would kiss you... but because the lack of some features a strong handshake is more appropiate for both of us,“ -- <span>yes, [that guy has started blogging.](http://blogs.msdn.com/windrago/)</span> </span>

<span>\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*</span> <span>Today I'll give a more precise definition of how JScript .NET determines whether one type is assignable to another. Remember, **assignable** is weaker than **promotable** -- promotable means that we are **guaranteed** that every instance of then source type is coercible to the target type.  With assignable, we care only if **some instance** of the source type is coercible to the target type. </span> <span><span>Next time, we'll finish up this **incredibly boring series** with the actual coercibility algorithm.  Then I think I'll take a short break and talk about some lighter topics before tackling overload resolution. </span></span>

<span></span>

<span>Again, suppose we have an assignment of an expression to a typed field. </span>

<span></span>

<span>Left Hand Side Expression = Right Hand Side Expression; </span>

<span></span>

<span>where the compiler can infer the type of the left hand side as LHT and the right hand side as RHT.  I'm going to be a little bit loose in my definition of assignable below in that sometimes I care about the **<span>type</span>** of the right hand expression, but sometimes we have more information than just the type and we want to determine whether the **<span>expression result</span>** is assignable.  It should be clear from the context which way I mean it. </span>

<span></span>

<span>Unfortunately, the current release of JScript .NET has a number of bugs -- it appears that the compiler does not actually enforce some of these rules at compile time when it could.  Assigning bad constant strings to enumerated types, for example, isn't caught until runtime.  Neither is the rule for double-\>single conversions.  Weird.  I'll mention those to the JScript .NET dev next time I see him.  </span>

<span></span>

<span>We start with three broad categories: </span>

  - <span>RH Expression is a **<span>compile-time constant. </span>**</span>
  - <span>**<span></span>**</span><span>RH Expression is an **<span>array literal. </span>**</span>
  - <span>**<span></span>**</span><span>RH Expression is **<span>something else. </span>**</span>

**<span>RHExp is a compile-time constant </span>**

<span> </span><span>This is clearly the simplest case.  We know not only the type but the value, so the question of whether RHT is assignable to LHT or not is irrelevant.  All we care about is whether this specific expression is assignable. </span>

<span></span>

  - <span>If constant RHExp is the **<span>name of a class</span>** then RHExp is assignable to LHT **<span>if and only if</span>** LHT is **<span>System.Type</span>** or **<span>System.Object</span>**. </span>
  - <span>If LHT is an **<span>enumerated type</span>** and RHExp is a **<span>string</span>** then RHExp is assignable to LHT **<span>if and only if </span>**the string names a member of that enumerated type. </span>
  - <span>If the constant RHExp is coercible to type LHT then it is assignable.  (More on this next time.) </span>
  - <span>If RHExp is a **<span>numeric literal</span>** and RHT is **<span>8-byte float</span>** and LHT is a **<span>4-byte float</span>** then RHExp is assignable to LHT. Yes, this is potentially lossy, but there's not much else we can do, so be careful\!  However… </span>
  - <span>If RHExp is a **<span>compile-time constant 8-byte float</span>** but NOT a numeric literal and LHT is a **<span>4-byte float</span>** then RHExp is assignable to LHT if and only if the string representation of the double and the string representation of that double downcast to single are identical.  (Oddly enough, this rule doesn't seem to be enforced by the compiler.  Looks like a bug.) </span><span></span>

**<span>RHExp is an array literal </span>**

<span></span> <span>Array literals are weird because they are treated as JScript arrays or system arrays depending on the context.  It matters very little what specifically RHExp is to determine assignment compatibility: </span>

<span></span>

  - <span>If LHT is **<span>System.Object</span>** or **<span>System.Array</span>** or **<span>JScript array</span>** then RHExp is assignable to LHT </span>
  - <span>If LHT is **<span>not some kind of array</span>** then RHExp is **<span>not</span>** assignable to LHT </span>
  - <span>If LHT is **<span>an array type of known rank and that rank is not 1</span>** then RHExp is **<span>not</span>** assignable to LHT </span>
  - <span>If LHT is **<span>an array of known type ElemType</span>** then RHExp is assignable to LHType **<span>if and only if every element in the array literal is assignable to ElemType.  </span>**</span><span></span>

<span>**RHExp is something else.** </span>

<span></span>

  - <span>If LHT is **<span>System.Object</span>** then RHT is assignable to LHT.</span>
  - <span>If LHT **<span>any primitive numeric type</span>** and RHT is 8-byte float then RHT is assignable to LHT.</span>
  - <span>If RHT is **<span>promotable</span>** to LHT then obviously RHT is assignable to LHT.</span>
  - <span>If LHT is a **<span>delegate</span>** and RHExp is a **<span>signature-compatible script function</span>** then RHExp is assignable to LHT.</span>
  - <span>If LHT is **<span>not any kind of array</span>** and RHT is **<span>JScript array</span>** then RHT is **<span>not</span>** assignable to LHT.</span>
  - <span>If LHT is **<span>System.Array</span>** and RHT is **<span>JScript array</span>** then RHT is assignable to LHT.  (I believe this gives a warning though.)</span>
  - <span>If LHT is an **<span>array type known to be rank 1</span>** and  RHT is **<span>JScript array</span>** then RHT is assignable to LHT.</span>
  - <span>If LHT is an **<span>array type known to be rank other than 1</span>** and  RHT is **<span>JScript array</span>** then RHT is not assignable to LHT.</span>
  - <span>If LHT is **<span>string</span>** then RHT is assignable -- just about everything can be converted to string.</span>
  - <span>If LHT is **<span>Boolean or any primitive numeric type</span>** and RHT is **<span>string</span>** then RHT is assignable to LHT. </span>
  - <span>If LHT is **<span>char</span>** and RHT is **<span>string</span>** then RHT is assignable to LHT. </span>
  - <span>If LHT is **<span>promotable</span>** to RHT then RHT is assignable to LHT. For instance, base classes are assignable to derived classes even though they are not promotable.</span>
  - <span>If LHT and RHT are both **<span>primitive numeric types</span>** then RHT is assignable to LHT.</span>

<span></span>

<span>Those last four may give a compile-time warning, as they're pretty dodgy.  </span>

</div>

</div>

