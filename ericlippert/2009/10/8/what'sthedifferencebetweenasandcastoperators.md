# What's the difference between "as" and "cast" operators?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2009 9:36:00 AM

-----

Most people will tell you that the difference between "(Alpha) bravo" and "bravo as Alpha" is that the former throws an exception if the conversion fails, whereas the latter returns null. Though this is correct, and this is the most obvious difference, it's not the only difference. There are pitfalls to watch out for here.

First off, since the result of the "as" operator can be null, the resulting type has to be one that takes a null value: either a reference type or a nullable value type. You cannot do "as int", that doesn't make any sense. If the argument isn't an int, then what should the return value be? The type of the "as" expression is always the named type so it needs to be a type that can take a null.

Second, [the cast operator, as I've discussed before, is a strange beast](http://blogs.msdn.com/ericlippert/archive/2009/03/19/representation-and-identity.aspx). It means two contradictory things: "check to see if this object really is of this type, throw if it is not" and "this object is not of the given type; find me an equivalent value that belongs to the given type". The latter meaning of the cast operator is not shared by the "as" operator. If you say

 

short s = (short)123;  
int? i = s as int?;

then you're out of luck. The "as" operator will not make the representation-changing conversions from short to nullable int like the cast operator would. Similarly, if you have class Alpha and unrelated class Bravo, with a user-defined conversion from Bravo to Alpha, then "(Alpha) bravo" will run the user-defined conversion, but "bravo as Alpha" will not. The "as" operator only considers reference, boxing and unboxing conversions.

And finally, of course the use cases of the two operators are superficially similar, but semantically quite different. A cast communicates to the reader "I am certain that this conversion is legal and I am willing to take a runtime exception if I'm wrong". The "as" operator communicates "I don't know if this conversion is legal or not; we're going to give it a try and see how it goes".

