# "For Each" vs. "for in"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/22/2003 5:19:00 PM

-----

While we're on the subject of semantic differences between seemingly similar syntaxes, let me just take this opportunity to quickly answer a frequently asked question: why doesn't for-in enumerate a collection?

A VB programmer is used to this printing out every item in a collection:

For Each Item In MyCollection  
    Print Item  
Next 

VB programmers introduced to JScript always make this mistake:

for (var item in myCollection)  
{  
    print(item);  
} 

and they are always surprised when this prints out a bunch of unexpected strings, or perhaps nothing at all.

The difference is quite simple.  In VBScript, For Each enumerates the **members of a collection**. In JScript, for in enumerates the **properties of an object**. In JScript if you have

var foo = new Object();  
foo.bar = 123;  
foo.baz = 456; 

then you can enumerate the properties:

for (var prop in foo)  
    print (prop + " : " + foo\[prop\]) 

JScript needs such a control flow structure because it has **expando objects** and **sparse arrays**. You might not know all the properties of an object or members of an associative array. It's not like VBScript where objects have fixed members and arrays are indexed by dense integer tuples. VBScript doesn't need this ability, so it doesn't have it.

Incidentally, in order to implement the JScript for in loop we needed to extend the functionality exposed by a dispatch object. Hence IDispatchEx, which gives the caller the ability to enumerate dispids and go from dispid back to name.

In JScript to enumerate members of a collection, use the Enumerator object:

for (var enumerator = new Enumerator(myCollection) ; \!enumerator.atEnd(); enumerator.moveNext())  
{  
    var item = enumerator.item();   
    // ... 

The reaction I get from people who have not seen object-oriented enumerators before is usually "yuck\!" This is an unfortunate reaction, as enumerator objects are extremely powerful. Unlike lexical For Each loops, enumerators are first-class objects. You can take out multiple enumerators on a collection, store them, pass them around, recycle them, all kinds of good stuff.

The semantics of the for in loop in JScript .NET are kind of a hodgepodge of both styles, with several interesting extensions. First off, if you pass an enumerator object itself to the JScript .NET for in loop, we enumerate it. If the argument is a JScript object (or a primitive convertible to a JScript object) then we enumerate its properties as JScript Classic does. If it is a CLR array, we enumerate its first dimension indices. If it is a collection, we fetch an enumerator and enumerate that. Otherwise, you've passed a non-enumerable object and we throw an exception

UPDATE: The sequel to this entry can be found [here](http://blogs.msdn.com/ericlippert/archive/2003/10/01/53134.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/01/53134.aspx").

