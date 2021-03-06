<div id="page">

# null is not false

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/26/2012 9:24:00 AM

-----

<div id="content">

<div class="mine">

The way you typically represent a "missing" or "invalid" value in C\# is to use the "null" value of the type. Every reference type has a "null" value; that is, the reference that does not actually refer to anything. And every "normal" value type has a corresponding "nullable" value type which has a null value.

The way these concepts are implemented is completely different. A reference is typically implemented behind the scenes as a 32 or 64 bit number. As we've discussed [previously in this space](http://blogs.msdn.com/b/ericlippert/archive/2009/02/17/references-are-not-addresses.aspx), that number should logically be treated as an "opaque" handle that only the garbage collector knows about, but in practice that number is the offset into the virtual memory space of the process that the referred-to object lives at, inside the managed heap. The number zero is reserved as the representation of null because the operating system reserves the first few pages of virtual memory as invalid, always. There is no chance that by some accident, the zero address is going to be a valid address in the heap.

By contrast, a nullable value type is simply an instance of the value type plus a Boolean that indicates whether the value is to be treated as a value, or as null. It's just a syntactic sugar for passing around a flag. This is because value types need not have any "special" value that has no other meaning; a byte has 256 possible values and every one of them is valid, so a nullable byte has to have some additional storage.

Some languages allow null values of value types or reference types, or both, to be implicitly treated as Booleans. In C, you can say:

<span class="code">int\* x = whatever();  
if (x) ...</span>

and that is treated as if you'd said "<span class="code">if (x \!= null)</span>". And similarly for nullable value types; in some languages a null value type is implicitly treated as "false".

The designers of C\# considered those features and rejected them. First, because treating references or nullable value types as Booleans is a confusing idiom and a potential rich source of bugs. And second, because semantically it seems presumptuous to automatically translate null -- which should mean "this value is missing" or "this value is unknown" -- to "this value is logically false".

In particular, we want to treat nullable bools as having three states: true, false and null, and not as having three states: true, false and different-kind-of-false. Treating null nullable Booleans as false leads to a number of oddities. Suppose we did, and suppose x is a nullable bool that is equal to null:

<span class="code">if (x)  
  Foo();  
if (\!x)  
  Bar();</span>

Neither Foo nor Bar is executed because "not null" is of course also null. (The answer to "what is the opposite of this unknown value?" is "an unknown value".) Does it not seem strange that <span class="code">x</span> and <span class="code">\!x</span> are both treated as false? Similarly, <span class="code">if (x | \!x)</span> would also be treated as false, which also seems bizarre.

The solution to the problem of these oddities is to avoid the problem in the first place, and not make nulls behave as though they were false.

Next time we'll look at a different aspect of truth-determining: just what is up with those "true" and "false" user-defined operators?

</div>

</div>

</div>

