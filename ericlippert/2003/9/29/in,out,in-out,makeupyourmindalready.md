# In, Out, In-Out, Make Up Your Mind Already

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/29/2003 1:09:00 PM

-----

I was talking about reference types vs. by-reference variables a while back ([here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx), [here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53005.aspx)and [here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx)). Recall that both JScript and VBScript have reference types (ie, objects) but JScript does not have by-reference variables. 

COM supports passing variable references around, but unfortunately the intersection of early-bound COM and late-bound IDispatch is a little bit goofy.  There are a few problems that you need to be aware of when you're trying to get VBScript code to talk to COM objects that expect to be passed references to variables. (Obviously attempting to do so from JScript is just a non-starter, as JScript will not pass variable references at all.) 

The most common problem happens when you are trying to call a method on a dispinterface where the vtable entry for the method looks something like this 

HRESULT MyFunction(\[in, out\] BSTR \* pbstrBlah ); 

Calling this method via VBScript like this produces a type mismatch error: 

Dim MyString  
MyString = "foo"  
MyObj.MyFunction MyString 

And calling it like this does not produce an error but also does not actually pass the value by reference: 

Dim MyString  
MyString = "foo"  
MyObj.MyFunction CStr(MyString) 

The latter behaviour is completely expected -- what you're passing is the output of a function, not a reference to a variable.  There is no reference available, so no value is filled in.  But why does the former fail?  

Well, VBScript does not know what the callee is expecting as far as types go.  That's what "late bound" means -- the callee has to do the work of determining how to suck the relevant data out of the variants passed to Invoke, and the callee has to somehow call the underlying vtable with the correct types.  So VBScript sees 

MyObj.MyFunction MyString 

and passes a reference to a variant.  **All variables are variants in VBScript. **

So why does VBScript produce a type mismatch error here?  **VBScript doesn't\!  The object produces the type mismatch error, which VBScript dutifully reports.** The object's implementation of Invoke calls the default implementation of Invoke provided for you by the type library implementation.  That thing says *"I've got a reference to a variant, and that variant is a string.  I need a reference to a string.  That's a type mismatch." *

Now, if* I* were designing such a system, I'd put some additional smarts into it that would handle this case.  Clearly from a reference to a variant that contains a string one can obtain a reference to a string -- just take the address of the string field of the variant\!  However, for reasons which are lost to the mists of time, the default implementation does not do that. 

My advice to you would therefore be 

1\)      If you want to write an object that can be easily used from script, **do not have any \[in, out\] parameters**, because JScript can't pass references.  
2\)      If you must have in/out parameters, **make them variants**, because VBScript can't pass any other kind of reference.  
3\)      If you must have nonvariant in/out parameters, **write some fixer-upper code for your **IDispatch** implementation which transforms byref variants pointing to strings into byref strings**. (Or whatever byref type you require.)  But if you do that, make sure you get it right.  Do not attempt to write your own IDispatch implementation, as there are many pitfalls (which I may discuss at another time). 

That's the most common problem I see.  The other common problem involves out parameters which are not in/out or out/retval parameters.  Just-plain-out parameters cause memory leaks.  Consider our earlier example: 

Dim MyString  
MyString = "foo"  
MyObj.MyFunction MyString 

Suppose MyFunction takes an in/out variant, and fills in the byref variant with the string "bar".  The implementor expects that something will come in and something will go out, and the rule is that the callee frees the coming-in memory before replacing it with the going-out memory.  The caller is then responsible for freeing the going-out value.  

But if MyFunction takes a just-plain-out variant then the callee does NOT free the incoming memory.  It assumes that the incoming memory is garbage because it has specifically been told that nothing is coming in.  

How does VBScript know whether the callee is in-out or out?  VBScript doesn't\!  Knowing that requires either compile time knowledge or a very expensive run-time lookup (the results of which are difficult to cache for the same reasons that dispatch ids are difficult to cache.) 

The practical result is that if you pass an object or string to a callee expecting an out variant, the string or object pointer will be overwritten without first being freed, and **the memory will leak**.  **You should ensure that you always pass empty variants if you must call an object method that takes an out parameter.**  Yes, this is a violation of the design principle that late-bound calls must have the same semantics as early bound calls, but unfortunately, that's just the way OLE Automation is and there's nothing we can do about it now.

