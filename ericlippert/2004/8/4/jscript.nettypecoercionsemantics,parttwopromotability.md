# JScript .NET Type Coercion Semantics, Part Two: Promotability

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/4/2004 5:49:00 PM

-----

Last time I gave some vague, high-level definitions of the type system concepts **promotable**, **assignable** and **coercible**. Today I'll give a more precise definition of **promotable**. The others I'll define more precisely later.

Suppose we have an assignment of an expression to a typed field. Left Hand Side Expression = Right Hand Side Expression; where the compiler can infer the type of the left hand side as LHT and the right hand side as RHT. **RHT is promotable to LHT if every member of RHT is coercible to LHT.** There are, as you'll see, some bugs and weirdnesses in JScript .NET's implementation of the promotability algorithm. This describes how it is, not how it should be\! In this list of rules there are some contradictions -- in those cases, earlier rules supercede later rules. **** General rules

  - If LHT **is the same type** as RHT then RHT is trivially promotable to LHT.
  - If LHT is **System.Object** and RHT **is a passed-by-reference type** then RHT is **not** promotable to LHT
  - If LHT is **System.Object** and RHT is anything other than a passed-by-reference type then RHT is promotable to LHT
  - If LHT is **any non-array type** and RHT is **an array type** then RHT is **not** promotable to LHT.
  - If LHT is **any array type** then see the algorithm below.
  - If LHT or RHT is an **enumerated type** then see the algorithm below.
  - If either RHT or LHT is a **class/interface** then see the algorithm below.
  - If RHT is the (single-member) **undefined** type or the (single member) **null** type then RHT is promotable to LHT regardless of what LHT is.
  - If LHT and RHT are both **primitive Boolean, numeric or date types** (Boolean, 1/2/4/8 byte signed/unsigned integers, 4/8 byte floats, decimal, datetime, timespan), see the algorithm below.
  - If LHT is a **JScript object wrapper for Boolean or string** and RHT is the **corresponding primitive type** then RHT is promotable to LHT.
  - If LHT is the **JScript object wrapper for number** and RHT is **promotable to a primitive 8-byte float** then RHT is promotable to LHT.
  - If LHT is the **primitive Boolean, string or datetime** and RHT is the **corresponding wrapper type** then RHT is promotable to LHT.
  - If LHT is a **primitive integer or float** and RHT is the **wrapper type for number** then RHT is promotable to LHT. (This one is an exception, as it is potentially lossy.)
  - If LHT has **defined an implicit cast operator** from RHT then RHT is promotable to LHT.
  - If RHT has **defined an implicit cast operator** to LHT then RHT is promotable to LHT.
  - Otherwise, RHT is not promotable to LHT.

Rules for the case where LHT is an array type

  - If LHT is **any array type** and RHT is **not an array type** then RHT is **not** promotable to LHT.
  - If LHT is **not a JScript array** and RHT is **a JScript array** then RHT is **not** promotable to LHT.
  - If LHT is **System.Array** then RHT is promotable to LHT, regardless of RHT.
  - If LHT is **any array type** and RHT is **System.Array** then RHT is **not** promotable to LHT.
  - If LHT is **a JScript array** and RHT **is any array known to be of rank 1** then RHT is promotable to LHT. Note that this is an exception, as it is potentially lossy or error-prone.
  - If LHT is of **known type and rank** and RHT is of **known type and rank**, and furthermore, the **ranks are equal** and RHT is **Element Type Compatible** with LHT then RHT is promotable to LHT.
  - Otherwise, RHT is not promotable to LHT.

Rules for the case where LHT or RHT is an enumerated type

  - If LHT and RHT are **different enumerated types** then RHT is not promotable to LHT.
  - If LHT is an **enumerated type** and RHT is a **primitive numeric type** promotable to the **underlying numeric type** of LHT them RHT is promotable to LHT.
  - If LHT is a **primitive numeric type** and RHT is an **enumerated type** whose **underlying numeric type** is promotable to LHT, then RHT is promotable to LHT.
  - If LHT is an **enumerated type** and RHT is **string** then RHT is promotable to LHT.
  - Otherwise, RHT is **not promotable** to LHT.

Note that the rules for enumerated types are exceptions; clearly there are always strings and numbers which are not coercible to a given enumerated type\! **** Rules for the case where RHT or LHT is a class/interface

  - If LHT is a **superclass** of RHT then RHT is promotable to LHT.  
  - If LHT is an **interface** **implemented** by RHT then RHT is promotable to LHT.
  - Otherwise, RHT is **not promotable** to LHT.

**** Rules for the case where RHT and LHT are both primitive types

  - If RHT is **Boolean or unsigned 1-byte integer** and LHT is **char, any** **integer, float, decimal, datetime or timespan** then RHT is promotable to LHT.
  - If RHT is **char or unsigned 2-byte integer** and LHT is **2-byte unsigned integer, any 4 or 8 byte integer, float, decimal, datetime or timespan** then RHT is promotable to LHT.
  - If RHT is **signed 1/2-byte integer** and LHT is **any signed integer, any float, decimal, datetime or timespan** then RHT is promotable to LHT. (This appears to be buggy -- a signed 2-byte integer should not be promotable to a signed 1-byte integer\! I'll look into it.)
  - If RHT is **signed 4-byte integer** and LHT is **signed 8-byte integer, 8-byte float, decimal, datetime or timespan** then RHT is promotable to LHT.
  - If RHT is **unsigned 4-byte integer** and LHT is **any 8-byte integer, 8-byte float, decimal, datetime or timespan** then RHT is promotable to LHT.
  - If RHT is **any 8-byte integer** and LHT is **decimal, datetime or timespan** then RHT is promotable to LHT.
  - If RHT is **any float** and LHT is **8-byte float or decimal** then RHT is promotable to LHT.
  - Otherwise, RHT is not promotable to LHT.

Next time, assignability.

