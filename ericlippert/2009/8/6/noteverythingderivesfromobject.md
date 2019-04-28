# Not everything derives from object

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/6/2009 9:45:00 AM

-----

I hear a lot of myths about C\#. Usually the myths have some germ of truth to them, like "[value types are always allocated on the stack](http://blogs.msdn.com/ericlippert/archive/2009/04/27/the-stack-is-an-implementation-detail.aspx)". If you replace "always" with "sometimes", then the incorrect mythical statement becomes correct.

One I hear quite frequently is "in C\# every type derives from object". Not true\!

First off, no pointer types derive from object, nor are any of them convertible to object. Unsafe pointer types are explicitly outside of the normal type rules for the language. (If you want to treat a pointer as a value type, convert it to System.IntPtr, and then you can use it as a value type.) We'll leave pointer types aside for the rest of this discussion.

A great many types do derive from object. All value types, including enums and nullable types, derive from object. All class, array and delegate types derive from object.

These are all the "concrete" types; every object instance that you actually encounter in the wild will be either null, or a class, delegate, array, struct, enum or nullable value type. So it would be correct, though tautological, to say that all object instances derive from object. But that wasn't the myth; the myth stated was that all *types* are derived from object.

Interface types, not being classes, are not derived from object. They are all *convertible* to object, to be sure, because we know that at runtime the instance will be a concrete type. But interfaces only derive from other interface types, and object is not an interface type.

"Open" type parameter types are also not derived from object. They are, to be sure, types. You can make a field whose type is a type parameter type. And since generic types and methods ultimately are only ever used at runtime when fully "constructed" with "concrete" type arguments, type parameter types are always *convertible* to object. (This is why you cannot make an IEnumerable\<void\*\> -- we require that the type argument be convertible to object.) Type parameter types are not derived from anything; they have an "effective base class", so that type arguments are constrained to be derived from the effective base class, but they themselves are not "derived" from anything.

The way to correct this myth is to simply replace "derives from" with "is convertible to", and to ignore pointer types: **every non-pointer type in C\# is *convertible* to object.**

