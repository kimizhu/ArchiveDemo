# For-in Revisited

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/1/2003 1:55:00 PM

-----

 

 

A while back [I was discussing](/ericlippert/archive/2003/09/22/53063.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/22/53063.aspx") the differences between VBScript's For-Each and JScript's for-in loops.  

A coworker asked me today whether there was any way to control the order in which the for-in loop enumerates the properties.  He wanted to get the list in alphabetical order for some reason. 

Unfortunately, we don't support that.  The specification says (ECMA 262 Revision 3 section 12.6.4): 

*The mechanics of enumerating the properties is implementation dependent. The order of enumeration is defined by the object. Properties of the object being enumerated may be deleted during enumeration. If a property that has not yet been visited during enumeration is deleted, then it will not be visited. If new properties are added to the object being enumerated during enumeration, the newly added properties are not guaranteed to be visited in the active enumeration. *

**

*Enumerating the properties of an object includes enumerating properties of its prototype, and the prototype of the prototype, and so on, recursively; but a property of a prototype is not enumerated if it is “shadowed” because some previous object in the prototype chain has a property with the same name. *

Our implementation enumerates the properties in the order that they were added.  (This also implies that properties added during the enumeration will be enumerated.) 

If you want to sort the keys then you'll have to do it the hard way -- which, fortunately, is not that hard.  Enumerate them in by-added order, add each to an array, and sort that array.  

var myTable = new Object();  
myTable\["blah"\] = 123;  
myTable \["abc"\] = 456  
myTable \[1234\] = 789;  
myTable \["def"\] = 346;  
myTable \[345\] = 566;  
  
var keyList = new Array();  
for(var prop in myTable)   
      keyList.push(prop);  
keyList.sort();  
  
for(var index = 0 ; index \< keyList.length ; ++index)  
      print(keyList\[index\] + " : " + myTable\[keyList\[index\]\]); 

This has the perhaps unfortunate property that it sorts numbers in alphabetical, not numeric order.  If that bothers you, then you can always pass a comparator to the sort method that sorts numbers however you'd like.

