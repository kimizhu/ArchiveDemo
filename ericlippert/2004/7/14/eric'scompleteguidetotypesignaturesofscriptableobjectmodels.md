# Eric's Complete Guide To Type Signatures Of Scriptable Object Models

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/14/2004 11:44:00 AM

-----

Moving away from [the problems of junior high school](http://blogs.msdn.com/ericlippert/archive/2004/07/12/181265.aspx "http://weblogs.asp.net/ericlippert/archive/2004/07/12/181265.aspx")…   Here's a question I've gotten many times over the years: **how do you design an object so that it can be easily called from both VBScript and JScript? **

 COM defines a fairly complex type system, and the script engines by design only support a subset of that type system.  It is certainly possible to create objects that can easily be called from C++ but which cannot be easily called from other languages such as VB6, VBScript, and especially JScript. 

 I've blogged about the *individual* issues many times; I've been asked for a document explaining it all in one place twice in the last month, so let me sum up.  I'll link back to the main articles for more details. 

 Note that this is fundamentally a list of "don'ts", not "dos".  Note also that there is plenty more to say about designing object models that are intuitive, discoverable, testable, documentable, extensible, etc, etc, etc.  Maybe I'll talk about that some day, but today I'm just concerned about the very basic question of what the implementor of method type signatures should avoid. 

 Numeric Types 

 VBScript can do arithmetic on VT\_I2 (short signed integer), VT\_I4 (long signed integer), VT\_R4 (single float), VT\_R8 (double float), VT\_CY (currency), and VT\_UI1 (unsigned byte) data.  **Eight-byte integers, unsigned integers, signed bytes and fixed-point decimals are not supported.** 

 JScript, by contrast, immediately converts bytes, shorts, singles, currencies and unsigned integers into either VT\_I4 or VT\_R8 (as appropriate), and then manipulates them as such. 

 In general, even if the script engines do not support operations on a particular type, you can still store the value and pass it around.  You can use script as "glue" to take, say, a VT\_DECIMAL from one object and pass it to another.  

 **If your object model uses fixed-point decimals (VT\_DECIMAL****), it will be hard to use it from script**.  Speaking of which, keep in mind that almost **all floating point arithmetic accrues errors.** If your object model depends on, say, using currencies (which are immune to floating point rounding issues at the cost of only supporting four decimal places) then JScript will likely defeat your purpose. Ideally, stick to doubles and longs.  

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/15/53000.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53000.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/53000.aspx") 

 Dates 

 VBScript and JScript use completely different code behind the scenes to keep track of dates.  **Both systems are fraught with "gotchas".**  If your code manipulates dates using the standard VT\_DATE used by COM, JScript will automatically translate them to its internal date format.  Generally speaking there are ways to make it all work, but the code can be slightly tricky. 

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/16/53013.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53013.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/16/53013.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx](http://blogs.msdn.com/ericlippert/archive/2003/10/06/53150.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/06/53150.aspx") 

 Object types 

 **No objects which speak interfaces other than IDispatch** **can be produced or consumed in VBScript or JScript**.  If you pass in a VT\_UNKNOWN, we try to turn it into a VT\_DISPATCH immediately.  Scriptable object models have to be fully dispatchable. 

 **Non-default dispatch interfaces are badness**.  JScript does not support non-default dispatches.  VBScript will use a non-default dispatch if given one, but exposes no way to obtain one from a default dispatch.  Don't write objects that support multiple disjoint dispatch interfaces. 

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/10/10/53188.aspx](http://blogs.msdn.com/ericlippert/archive/2003/10/10/53188.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/10/53188.aspx") 

 Missing arguments, missing information 

 VBScript supports missing arguments.  You can call a method like this: 

 x = foo.bar(123, , 456) 

 and VBScript will pass a VT\_ERROR for the parameter with the error field set to PARAMNOTFOUND.  The callee is then responsible for handling the problem, filling in a sensible default value, whatever. 

 **JScript does not support missing arguments**.  Object models where there are methods that take dozens of arguments where some of them are expected to be missing are hard to use from JScript.  (And they are probably badly designed methods just on first principles\!)  

 Neither JScript nor VBScript supports **named arguments**, so that's right out too. 

 A common alternative to passing a missing argument is to pass Nothing, Null, or Empty in VBScript, null or undefined in JScript.  Null and null pass VT\_NULL, Empty and undefined pass VT\_EMPTY, and Nothing passes a VT\_DISPATCH with no value dispatch object pointer.  There's no easy way in JScript to pass a "null" object.  I consider this to be a design flaw in JScript, but we're stuck with it now.  Designers of good scriptable object models might consider treating Null and Nothing the same to make it easier to use from JScript. 

 Another related issue is the "empty/null string" problem.  In C, an empty string and a null string may be treated differently.  Not so in VB6, VBScript and JScript; they use the convention that a **null string and empty string are equivalent**.  But VB can call Win32 APIs which do not use this convention, so VB6 and VBScript allow you to force the runtime to pass an empty string or a null string.  **JScript does not have this feature.** 

 A dispatchable COM object model which makes a distinction between null and empty strings is almost certainly buggy.  **Don't go there**. 

 For more information, see: 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/12/52976.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/12/52976.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/12/52976.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/30/53120.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/10/01/53128.aspx](http://blogs.msdn.com/ericlippert/archive/2003/10/01/53128.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/01/53128.aspx") More On Parameter Passing 

 "**Out" parameters are badness** -- do not write methods that have out parameters in scriptable object models.  JScript does not support out parameters at all.  There are memory leaking scenarios for VBScript.  

 **"In-out" parameters are also badness**.  JScript does not support in-out parameters at all.  VBScript always passes VT\_VARIANT | VT\_BYREF, and the default implementation of IDispatch::Invoke will give a type mismatch if the method is expecting a **hard-typed** byref parameter.  You can either write your own coercion logic, or make the argument a variant, but both are tricky. Best to avoid it entirely -- don't use out or in-out parameters. 

 "Out-retval" values are not really parameters -- they are return values -- so they are fine. 

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/09/15/53005.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53005.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/53005.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/09/29/53117.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/29/53117.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/29/53117.aspx") 

 Arrays 

 **Arrays are badness**.  JScript has very poor support for VB-style arrays.  Multi-dimensional vb-style arrays are particularly hard to deal with in JScript.  Try to not write object models that take arrays as parameters, or which can only return arrays.  (This is also a good idea because it may avoid the perf cost of copying around large arrays.) 

 VB6 can create arrays with arbitrary index ranges, such as foo(10 to 20, 4 to 6).  VBScript can *consume* such arrays but cannot *produce* them -- all VBScript-produced arrays have lower bounds of zero. 

 **The script engines only support arrays of variants**.  An array of bytes can be converted to a string by the underlying operating system, but that's pretty hacked up. 

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/22/53061.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/22/53061.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/22/53061.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2003/09/22/53069.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/22/53069.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/22/53069.aspx")  
[http://blogs.msdn.com/ericlippert/archive/2004/05/25/141525.aspx](http://blogs.msdn.com/ericlippert/archive/2004/05/25/141525.aspx "http://blogs.msdn.com/ericlippert/archive/2004/05/25/141525.aspx") 

 Collections 

 Both VBScript and JScript support iterating collections that implement IEnumVARIANT.  **No other enumerators are supported.** 

 For more information see 

 [http://blogs.msdn.com/ericlippert/archive/2003/09/22/53063.aspx](http://blogs.msdn.com/ericlippert/archive/2003/09/22/53063.aspx "http://weblogs.asp.net/ericlippert/archive/2003/09/22/53063.aspx") 

 Method names 

 Use straight A-Z 0-9 characters in object model names.  Names with underbars are badness, just on general principles.  They're too easily confused with the line continuation character in VB, and just look ugly.  Avoid. 

 Names which collide with JScript or VBScript reserved words or built-in object model elements are also a bad idea.  Don't go calling your methods InStr or Math.cos, you'll just confuse people\! 

 For more information see 

 [http://weblogs.asp.net/ericlippert/archive/2004/06/10/152831.aspx](http://blogs.msdn.com/ericlippert/archive/2004/06/10/152831.aspx "http://weblogs.asp.net/ericlippert/archive/2004/06/10/152831.aspx") 

 I'm sure there's stuff I'm forgetting.  I reserve the right to update this list in the future\!

