<div id="page">

# For-in Revisited

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/1/2003 1:55:00 PM

-----

<div id="content">

<span> </span>

<span> </span>

<span>A while back [I was discussing](/ericlippert/archive/2003/09/22/53063.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/22/53063.aspx") the differences between VBScript's </span><span>For-Each</span><span> and JScript's </span><span>for-in</span><span> loops.  </span>

<span></span>

<span>A coworker asked me today whether there was any way to control the order in which the </span><span>for-in</span><span> loop enumerates the properties.  He wanted to get the list in alphabetical order for some reason. </span>

<span></span>

<span>Unfortunately, we don't support that.  The specification says (ECMA 262 Revision 3 section 12.6.4): </span>

<span></span>

*<span>The mechanics of enumerating the properties is implementation dependent. The order of enumeration is defined by the object. Properties of the object being enumerated may be deleted during enumeration. If a property that has not yet been visited during enumeration is deleted, then it will not be visited. If new properties are added to the object being enumerated during enumeration, the newly added properties are not guaranteed to be visited in the active enumeration. </span>*

*<span></span>*

*<span>Enumerating the properties of an object includes enumerating properties of its prototype, and the prototype of the prototype, and so on, recursively; but a property of a prototype is not enumerated if it is “shadowed” because some previous object in the prototype chain has a property with the same name. </span>*

<span></span>

<span>Our implementation enumerates the properties in the order that they were added.  (This also implies that properties added during the enumeration will be enumerated.) </span>

<span></span>

<span>If you want to sort the keys then you'll have to do it the hard way -- which, fortunately, is not that hard.  Enumerate them in by-added order, add each to an array, and sort that array. </span><span> </span>

<span>var myTable = new Object();  
</span><span>myTable\["blah"\] = 123;  
</span><span>myTable \["abc"\] = 456  
</span><span>myTable \[1234\] = 789;  
</span><span>myTable \["def"\] = 346;  
</span><span>myTable \[345\] = 566;  
</span><span>  
var keyList = new Array();  
</span><span>for(var prop in myTable)   
</span><span>      keyList.push(prop);  
</span><span>keyList.sort();  
</span><span>  
for(var index = 0 ; index \< keyList.length ; ++index)  
</span><span>      print(keyList\[index\] + " : " + myTable\[keyList\[index\]\]); </span>

<span></span>

<span>This has the perhaps unfortunate property that it sorts numbers in alphabetical, not numeric order.  If that bothers you, then you can always pass a comparator to the sort method that sorts numbers however you'd like. </span>

</div>

</div>

