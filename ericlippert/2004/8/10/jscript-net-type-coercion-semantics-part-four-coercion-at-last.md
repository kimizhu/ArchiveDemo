<div id="page">

# JScript .NET Type Coercion Semantics, Part Four: Coercion at last

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/10/2004 10:56:00 AM

-----

<div id="content">

<span>Before I get going, a couple notable milestones.  </span> <span>First, this is **post number 200**\!  Who would have believed that I'd have so much to ramble on about?  ("Anyone who knows you" would be the correct answer to that rhetorical question I suppose.)  </span>

<span></span> <span>Second, as of Sunday I am now over (cue Dr. Evil) **<span>one billion seconds</span>** old.  Happy gigasecond to me, happy gigasecond to me… </span>

<span></span> <span>OK, enough frivolity.  Back to boring facts about JScript .NET's coercibility algorithm.</span> <span>\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* </span>

<span></span> <span>Today I'll finish up and give a more precise definition of how JScript .NET determines whether a particular value is coercible to a given type. Then we'll get back into lighter topics. </span>

<span></span> <span>Suppose we have a value V which we wish to coerce to type T **<span>without data loss or error</span>**.  For clarity I'll split the algorithm up into two sections, one for "special" types like classes, arrays and enumerations, and one for "primitive value" types like integers and strings. </span>

<span></span> <span>Suppose T is a reference, enumerated, array, etc, type: </span>

<span></span>

  - <span>If V is **<span>null or undefined</span>** then V is coercible to T irrespective of T.</span>
  - <span>If T is **<span>System.Object</span>** then V is coercible to T irrespective of V.</span>
  - <span>If T is a **<span>class</span>** and V is an **<span>instance</span>** of T (which includes being an instance of a subclass of T) then V is coercible to T.  </span>
  - <span>If T is an **<span>interface</span>** and V **<span>implements</span>** T then V is coercible to T.</span>
  - <span>If T is a **<span>non-JScript array of known element type E</span>** and V is a **<span>JScript array</span>** then V is coercible to T **<span>if and only if</span>** every element of V is coercible to E.</span>
  - <span>If T is a **<span>JScript array</span>** and V is a **<span>rank-one non-JScript array</span>** then V is coercible to T.</span>
  - <span>If T is an **<span>enumerated type</span>** and  V is a **<span>member of enumerated type E</span>** then V is coercible to T **<span>if and only if</span>** T and E are the same type.</span>
  - <span>If T is an **<span>enumerated type</span>** and V is a **<span>string</span>** and then V is coercible to T **<span>if and only if</span>** V names a member of T.</span>
  - <span>If T is an **<span>enumerated type of implementation type E</span>** and V is **<span>not a member of an enumerated type</span>** then V is coercible to T **<span>if and only if</span>** V is coercible to E.</span>
  - <span>If T is the **<span>System.Type</span>** type and V is a **<span>class name</span>** (not an *<span>instance</span>* of a class but actually the name of a class) then V is coercible to T.</span>
  - <span>If T is a **<span>delegate type</span>** and V is a **<span>function object</span>** (including a closure) then V is coercible to T **<span>if and only if</span>** V has the same function signature as the delegate.</span>
  - <span>If T **<span>defines an appropriate type coercion function</span>** then V is coercible to T **<span>if and only if</span>** the function call doesn't throw an exception.</span>

<span></span> <span>Now let's consider the primitive type coercion rules. </span>

<span></span> <span>JScript .NET supports several numeric types: 8, 16, 32 and 64 bit integers in both signed and unsigned flavours, 32 and 64 bit floats and a decimal type.  JScript .NET also supports a "char" type which is an unsigned 16 bit integer that holds a single Unicode UTF-16 value.  JScript .NET also supports a scalar Date/Time type which is treated the same as a signed 64 bit integer unless otherwise noted. </span>

<span></span> <span>Note that **<span>data loss is allowed when coercing anything to Boolean and when coercing strings to numbers. </span>**</span>

<span></span>

  - <span>If T **<span>is the type</span>** of V then V is coercible to T.  (Duh\!)</span>
  - <span>If V is **<span>undefined or null</span>** then V is coercible to T irrespective of T.  (FYI, coercing V to Boolean goes to false.  Coercing V to any numeric type goes to 0. Coercing V to string goes to "".)</span>
  - <span>If V is **<span>Boolean</span>** then V is coercible to T irrespective of T.  (FYI, coercing a V to a numeric type goes to 0 or 1, coercing V to string goes to "true" or "false".)</span>
  - <span>If V is **<span>char</span>** then V is coercible to T irrespective of T. (FYI, coercing V to Boolean goes to false if char is \\u0000, true otherwise.  Coercing V to numeric goes to the UTF-16 value. Coercing V to string goes to a single-character string.)</span>
  - <span>If T is **<span>any numeric type</span>** and V is a member of **<span>any numeric type</span>** then V is coercible to T if and only if V can be coerced to T without overflow or loss of precision.  For instance, if V were the unsigned 8-byte integer 300 then V could be coerced to an unsigned 2-byte integer regardless of the fact that the 8-byte unsigned integer type is not promotable to the 2 byte integer type. Note that if V is, say, the 8-byte float value 0.1 then this is not coercible to a 4-byte float as there will be a loss of precision.  (0.5 would be coercible.)  Recall that the assignability algorithm has a special case to ensure that compile-time constants inferred to be 64 bit floats are assignable to 32 bit floats even if they are not coercible.</span>
  - <span>If T is **<span>string</span>** and V is of **<span>any numeric type</span>** then V is coercible to T.  (JScript .NET uses the standard number-to-string routine.  If V is a Date/Time scalar then it uses the appropriate date string.)</span>
  - <span>If T is **<span>Boolean</span>** and V is of **<span>any numeric type</span>** then V is coercible to T.  (If V is zero or NaN then it goes to false, otherwise it goes to true.)</span>
  - <span>If T is **<span>Boolean</span>** and V is a **<span>string</span>** then V is coercible to T.  (If V is "" then it goes to false, otherwise true.)</span>
  - <span>If T is a **<span>date type</span>** and V is a **<span>string</span>** then V is coercible to T **<span>if and only if</span>** V is parsable as a date.</span>
  - <span>If T is **<span>char</span>** and V is a **<span>string</span>** then V is coercible to T **<span>if and only if</span>** V is a **<span>single-character string.</span>**</span>
  - <span>If T is a **<span>numeric type</span>** and V is a **<span>string</span>** **<span>that can be parsed as type T</span>** then V is coercible to T </span>
  - <span>If T is a **<span>numeric type</span>** and V is a **<span>string</span>** **<span>that cannot be parsed as type T</span>** then V is coercible to T **<span>if and only if</span>** V is parsable to an 8-byte float and the resulting float value is coercible to T.</span>

<span></span> <span>If none of the above apply, V is not coercible to T, and will likely cause a type mismatch exception. </span>

<span></span>

<span></span>

</div>

</div>

