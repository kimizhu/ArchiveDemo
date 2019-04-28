<div id="page">

# "For Each" vs. "for in"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/22/2003 5:19:00 PM

-----

<div id="content">

<div class="mine">

While we're on the subject of semantic differences between seemingly similar syntaxes, let me just take this opportunity to quickly answer a frequently asked question: why doesn't <span class="code">for-in</span> enumerate a collection?

A VB programmer is used to this printing out every item in a collection:

<span class="code">For Each Item In MyCollection  
    Print Item  
Next </span>

VB programmers introduced to JScript always make this mistake:

<span class="code">for (var item in myCollection)  
{  
    print(item);  
} </span>

and they are always surprised when this prints out a bunch of unexpected strings, or perhaps nothing at all.

The difference is quite simple.  In VBScript, <span class="code">For Each</span> enumerates the **members of a collection**. In JScript, <span class="code">for in</span> enumerates the **<span>properties of an object</span>**. In JScript if you have

<span class="code">var foo = new Object();  
foo.bar = 123;  
foo.baz = 456; </span>

then you can enumerate the properties:

<span class="code">for (var prop in foo)  
    print (prop + " : " + foo\[prop\]) </span>

JScript needs such a control flow structure because it has **<span>expando objects</span>** and **<span>sparse arrays</span>**. You might not know all the properties of an object or members of an associative array. It's not like VBScript where objects have fixed members and arrays are indexed by dense integer tuples. VBScript doesn't need this ability, so it doesn't have it.

Incidentally, in order to implement the JScript <span class="code">for in</span> loop we needed to extend the functionality exposed by a dispatch object. Hence <span class="code">IDispatchEx</span>, which gives the caller the ability to enumerate dispids and go from dispid back to name.

In JScript to enumerate members of a collection, use the <span class="code">Enumerator</span> object:

<span class="code">for (var enumerator = new Enumerator(myCollection) ; \!enumerator.atEnd(); enumerator.moveNext())  
{  
    var item = enumerator.item();   
    // ... </span>

The reaction I get from people who have not seen object-oriented enumerators before is usually "yuck\!" This is an unfortunate reaction, as enumerator objects are extremely powerful. Unlike lexical <span class="code">For Each</span> loops, enumerators are first-class objects. You can take out multiple enumerators on a collection, store them, pass them around, recycle them, all kinds of good stuff.

The semantics of the <span class="code">for in</span> loop in JScript .NET are kind of a hodgepodge of both styles, with several interesting extensions. First off, if you pass an enumerator object itself to the JScript .NET <span class="code">for in</span> loop, we enumerate it. If the argument is a JScript object (or a primitive convertible to a JScript object) then we enumerate its properties as JScript Classic does. If it is a CLR array, we enumerate its first dimension indices. If it is a collection, we fetch an enumerator and enumerate that. Otherwise, you've passed a non-enumerable object and we throw an exception

UPDATE: The sequel to this entry can be found [here](http://blogs.msdn.com/ericlippert/archive/2003/10/01/53134.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/01/53134.aspx").

</div>

</div>

</div>

