<div id="page">

# In, Out, In-Out, Make Up Your Mind Already

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/29/2003 1:09:00 PM

-----

<div id="content">

<span></span>

<div>

<span>I was talking about reference types vs. by-reference variables a while back ([here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/52996.aspx), [here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53005.aspx) and [here](http://blogs.msdn.com/ericlippert/archive/2003/09/15/53006.aspx)). Recall that both JScript and VBScript have reference types (ie, objects) but JScript does not have by-reference variables. </span>

<span></span>

<span>COM supports passing variable references around, but unfortunately the intersection of early-bound COM and late-bound </span><span>IDispatch</span><span> is a little bit goofy.  There are a few problems that you need to be aware of when you're trying to get VBScript code to talk to COM objects that expect to be passed references to variables. (Obviously attempting to do so from JScript is just a non-starter, as JScript will not pass variable references at all.) </span>

<span></span>

<span>The most common problem happens when you are trying to call a method on a dispinterface where the vtable entry for the method looks something like this </span>

<span></span>

<span>HRESULT MyFunction(\[in, out\] BSTR \* pbstrBlah ); </span>

<span></span>

<span>Calling this method via VBScript like this produces a type mismatch error: </span>

<span></span>

<span>Dim MyString  
</span><span>MyString = "foo"  
</span><span>MyObj.MyFunction MyString </span>

<span></span>

<span>And calling it like this does not produce an error but also does not actually pass the value by reference: </span>

<span></span>

<span>Dim MyString  
</span><span>MyString = "foo"  
</span><span>MyObj.MyFunction CStr(MyString) </span>

<span></span>

<span>The latter behaviour is completely expected -- what you're passing is the output of a function, not a reference to a variable.  There is no reference available, so no value is filled in.  But why does the former fail?  </span>

<span></span>

<span>Well, VBScript does not know what the callee is expecting as far as types go.  That's what "late bound" means -- the callee has to do the work of determining how to suck the relevant data out of the variants passed to Invoke, and the callee has to somehow call the underlying vtable with the correct types.  So VBScript sees </span>

<span></span>

<span>MyObj.MyFunction MyString </span>

<span></span>

<span>and passes a reference to a variant.  **<span>All variables are variants in VBScript. </span>**</span>

<span></span>

<span>So why does VBScript produce a type mismatch error here?  **<span>VBScript doesn't\!  The object produces the type mismatch error, which VBScript dutifully reports.</span>** The object's implementation of Invoke calls the default implementation of Invoke provided for you by the type library implementation.  That thing says *<span>"I've got a reference to a variant, and that variant is a string.  I need a reference to a string.  That's a type mismatch." </span>*</span>

<span></span>

<span>Now, if*<span> I</span>* were designing such a system, I'd put some additional smarts into it that would handle this case.  Clearly from a reference to a variant that contains a string one can obtain a reference to a string -- just take the address of the string field of the variant\!  However, for reasons which are lost to the mists of time, the default implementation does not do that. </span>

<span></span>

<span>My advice to you would therefore be </span>

<span></span>

<span>1)</span><span>      </span><span>If you want to write an object that can be easily used from script, **<span>do not have any \[in, out\] parameters</span>**, because JScript can't pass references.  
</span><span>2)</span><span>      </span><span>If you must have in/out parameters, **<span>make them variants</span>**, because VBScript can't pass any other kind of reference.  
</span><span>3)</span><span>      </span><span>If you must have nonvariant in/out parameters, **<span>write some fixer-upper code for your </span>**</span><span>IDispatch</span>**<span> implementation which transforms byref variants pointing to strings into byref strings</span>**<span>. (Or whatever byref type you require.)  But if you do that, make sure you get it right.  Do not attempt to write your own </span><span>IDispatch</span><span> implementation, as there are many pitfalls (which I may discuss at another time). </span>

<span></span>

<span>That's the most common problem I see.  The other common problem involves out parameters which are not in/out or out/retval parameters.  Just-plain-out parameters cause memory leaks.  Consider our earlier example: </span>

<span></span>

<span>Dim MyString  
</span><span>MyString = "foo"  
</span><span>MyObj.MyFunction MyString </span>

<span></span>

<span>Suppose </span><span>MyFunction</span><span> takes an in/out variant, and fills in the byref variant with the string "bar".  The implementor expects that something will come in and something will go out, and the rule is that the callee frees the coming-in memory before replacing it with the going-out memory.  The caller is then responsible for freeing the going-out value.  </span>

<span></span>

<span>But if </span><span>MyFunction</span><span> takes a just-plain-out variant then the callee does NOT free the incoming memory.  It assumes that the incoming memory is garbage because it has specifically been told that nothing is coming in.  </span>

<span></span>

<span>How does VBScript know whether the callee is in-out or out?  VBScript doesn't\!  Knowing that requires either compile time knowledge or a very expensive run-time lookup (the results of which are difficult to cache for the same reasons that dispatch ids are difficult to cache.) </span>

<span></span>

<span>The practical result is that if you pass an object or string to a callee expecting an out variant, the string or object pointer will be overwritten without first being freed, and **<span>the memory will leak</span>**.  **<span>You should ensure that you always pass empty variants if you must call an object method that takes an out parameter.</span>**  Yes, this is a violation of the design principle that late-bound calls must have the same semantics as early bound calls, but unfortunately, that's just the way OLE Automation is and there's nothing we can do about it now.</span>

</div>

</div>

</div>

