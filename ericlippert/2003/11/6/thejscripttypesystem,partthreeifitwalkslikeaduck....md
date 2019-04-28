# The JScript Type System, Part Three: If It Walks Like A Duck...

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/6/2003 7:05:00 PM

-----

A reader asks "can you explain the logic that a string is not always a String but a regexp is always a RegExp? What is the recommended way of determining if a value is a string?" 

 

 

Indeed, you are correct:

 

 

print(/foo/ instanceof RegExp);             // true

print(new RegExp("foo") instanceof RegExp); // true

print("bar" instanceof String);             // false

print(new String("bar") instanceof String); // true

print(typeof("bar"));                       // string

print(typeof(new String("bar")));           // object

 

 

Why's that?

 

 

First off, the question about strings.  In JScript there is this bizarre feature where primitive values -- Booleans, strings, numbers -- can be "wrapped up" into objects.  Doing so leads to some bizarre situations.  First off, as you note, the type of a wrapped primitive is always an object type, not a primitive type.  Also, we use object equality, not value equality.   

 

 

print(new String("bar") == new String("bar")); // false 

 

 

I highly recommend against using wrapped primitives.  Why do they exist?  Well, the reasoning has kind of been lost in the mists of time, but one good reason is to make the prototype inheritance system consistent.  If "bar" is not an object then how is it possible to say 

 

 

print("bar".toUpperCase()); 

 

 

? Well, actually, from the point of view of the specification, this is just a syntactic sugar for

 

 

print((new String("bar")).toUpperCase()); 

 

 

Now, of course as an implementation detail we do not **actually** cons up a new object every time you call a property on a value type\!  That would be a performance nightmare.  The runtime engine is smart enough to realize that it has a value type and that it ought to pass it as the "this" object to the appropriate method on String.prototype and everything just kind of works out.

 

 

This also explains why it is possible to stick properties onto value types that magically disappear.  When you say

 

 

var bar = "bar";

bar.hello = "hello";

print(bar.hello); // nada\!

 

 

of course what is happening is logically equivalent to:

 

 

var bar = "bar";

(new String(bar)).hello = "hello";

print((new String(bar)).hello); // nada\!

 

 

See, the magical temporary object is just that -- magical and temporary.  Once you've used it, poof, it disappears.

 

 

But this magical temporary object does not appear when the typeof or instanceof operators are involved.  The instanceof operator says "hey, this thing isn't even an object, so it can't possibly be an instance of anything".  For both consistency and usability, it would have been nice if "bar" instanceof String created a temporary object and hence said yes, it is an instance of String.  But for whatever reason, that's not the specification that the committee came up with.

 

 

Second, your question about regular expressions is easily answered now that we know what is going on with strings.  The difference between regular expressions and strings is that **regular expressions are not primitives**.  Just because you have the ability to express a regular expression as a *literal* does not mean that it is a *primitive*\!  That thing is always an object, so there is no behaviour difference between the compile-time-literal syntax and the runtime syntax.

 

 

Third, your question about how to determine whether something is a string is surprisingly tricky.  If typeof returns "string" then obviously it is a string, end of story.  But what if typeof returns "object" -- how can you tell if that thing is a wrapped string?   

 

 

It's not easy.   instanceof String doesn't tell you whether that thing is a string, it tells you whether String.prototype is on the prototype chain.  There's nothing stopping you from saying

 

 

function MyString() {}

MyString.prototype = String.prototype;

var s = new MyString();

print(s.constructor == String);            // true

print(s instanceof String);                // true

print(String.prototype.isPrototypeOf(s));  // true

 

 

So now what are you going to do?  JScript is excessively dynamic\!  Basically you can't rely on any object being what it says it is.  JScript forces people to be operationalists.  (Operationalism is the philosophical belief that if it walks like a duck and quacks like a duck, it *is* a duck.)  In the face of the kind of weirdness described above, all you can do is try to use the thing like a string, and if it acts like a string, it s a string.

