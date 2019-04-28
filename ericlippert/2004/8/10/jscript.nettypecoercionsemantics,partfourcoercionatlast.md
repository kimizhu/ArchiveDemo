# JScript .NET Type Coercion Semantics, Part Four: Coercion at last

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/10/2004 10:56:00 AM

-----

Before I get going, a couple notable milestones.   First, this is **post number 200**\!  Who would have believed that I'd have so much to ramble on about?  ("Anyone who knows you" would be the correct answer to that rhetorical question I suppose.)  

 Second, as of Sunday I am now over (cue Dr. Evil) **one billion seconds** old.  Happy gigasecond to me, happy gigasecond to me… 

 OK, enough frivolity.  Back to boring facts about JScript .NET's coercibility algorithm. \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 

 Today I'll finish up and give a more precise definition of how JScript .NET determines whether a particular value is coercible to a given type. Then we'll get back into lighter topics. 

 Suppose we have a value V which we wish to coerce to type T **without data loss or error**.  For clarity I'll split the algorithm up into two sections, one for "special" types like classes, arrays and enumerations, and one for "primitive value" types like integers and strings. 

 Suppose T is a reference, enumerated, array, etc, type: 

  - If V is **null or undefined** then V is coercible to T irrespective of T.
  - If T is **System.Object** then V is coercible to T irrespective of V.
  - If T is a **class** and V is an **instance** of T (which includes being an instance of a subclass of T) then V is coercible to T.  
  - If T is an **interface** and V **implements** T then V is coercible to T.
  - If T is a **non-JScript array of known element type E** and V is a **JScript array** then V is coercible to T **if and only if** every element of V is coercible to E.
  - If T is a **JScript array** and V is a **rank-one non-JScript array** then V is coercible to T.
  - If T is an **enumerated type** and  V is a **member of enumerated type E** then V is coercible to T **if and only if** T and E are the same type.
  - If T is an **enumerated type** and V is a **string** and then V is coercible to T **if and only if** V names a member of T.
  - If T is an **enumerated type of implementation type E** and V is **not a member of an enumerated type** then V is coercible to T **if and only if** V is coercible to E.
  - If T is the **System.Type** type and V is a **class name** (not an *instance* of a class but actually the name of a class) then V is coercible to T.
  - If T is a **delegate type** and V is a **function object** (including a closure) then V is coercible to T **if and only if** V has the same function signature as the delegate.
  - If T **defines an appropriate type coercion function** then V is coercible to T **if and only if** the function call doesn't throw an exception.

 Now let's consider the primitive type coercion rules. 

 JScript .NET supports several numeric types: 8, 16, 32 and 64 bit integers in both signed and unsigned flavours, 32 and 64 bit floats and a decimal type.  JScript .NET also supports a "char" type which is an unsigned 16 bit integer that holds a single Unicode UTF-16 value.  JScript .NET also supports a scalar Date/Time type which is treated the same as a signed 64 bit integer unless otherwise noted. 

 Note that **data loss is allowed when coercing anything to Boolean and when coercing strings to numbers. **

  - If T **is the type** of V then V is coercible to T.  (Duh\!)
  - If V is **undefined or null** then V is coercible to T irrespective of T.  (FYI, coercing V to Boolean goes to false.  Coercing V to any numeric type goes to 0. Coercing V to string goes to "".)
  - If V is **Boolean** then V is coercible to T irrespective of T.  (FYI, coercing a V to a numeric type goes to 0 or 1, coercing V to string goes to "true" or "false".)
  - If V is **char** then V is coercible to T irrespective of T. (FYI, coercing V to Boolean goes to false if char is \\u0000, true otherwise.  Coercing V to numeric goes to the UTF-16 value. Coercing V to string goes to a single-character string.)
  - If T is **any numeric type** and V is a member of **any numeric type** then V is coercible to T if and only if V can be coerced to T without overflow or loss of precision.  For instance, if V were the unsigned 8-byte integer 300 then V could be coerced to an unsigned 2-byte integer regardless of the fact that the 8-byte unsigned integer type is not promotable to the 2 byte integer type. Note that if V is, say, the 8-byte float value 0.1 then this is not coercible to a 4-byte float as there will be a loss of precision.  (0.5 would be coercible.)  Recall that the assignability algorithm has a special case to ensure that compile-time constants inferred to be 64 bit floats are assignable to 32 bit floats even if they are not coercible.
  - If T is **string** and V is of **any numeric type** then V is coercible to T.  (JScript .NET uses the standard number-to-string routine.  If V is a Date/Time scalar then it uses the appropriate date string.)
  - If T is **Boolean** and V is of **any numeric type** then V is coercible to T.  (If V is zero or NaN then it goes to false, otherwise it goes to true.)
  - If T is **Boolean** and V is a **string** then V is coercible to T.  (If V is "" then it goes to false, otherwise true.)
  - If T is a **date type** and V is a **string** then V is coercible to T **if and only if** V is parsable as a date.
  - If T is **char** and V is a **string** then V is coercible to T **if and only if** V is a **single-character string.**
  - If T is a **numeric type** and V is a **string** **that can be parsed as type T** then V is coercible to T 
  - If T is a **numeric type** and V is a **string** **that cannot be parsed as type T** then V is coercible to T **if and only if** V is parsable to an 8-byte float and the resulting float value is coercible to T.

 If none of the above apply, V is not coercible to T, and will likely cause a type mismatch exception.

