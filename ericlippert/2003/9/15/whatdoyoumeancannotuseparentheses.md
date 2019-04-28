# What do you mean "cannot use parentheses?"

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/15/2003 4:09:00 PM

-----

Every now and then someone will ask me what the VBScript error message "Cannot use parentheses when calling a Sub" means. I always smile when I hear that question. I tell people that the error means that you CANNOT use PARENTHESES when CALLING a SUB -- which word didn't you understand?

Of course, there is a reason why people ask, even though the error message is perfectly straightforward. Usually what happens is someone writes code like this:

Result = MyFunc(MyArg)  
MySub(MyArg)

and it works just fine, so they then write MyOtherSub(MyArg1, MyArg2) only to get the above error.

Here's the deal: **parentheses mean several different things in VB and hence in VBScript**. They mean:

1\) Evaluate a subexpression before the rest of the expression: Average = (First + Last) / 2  
2\) Dereference the index of an array: Item = MyArray(Index)  
3\) Call a function or subroutine: Limit = UBound(MyArray)  
4\) Pass an argument which would normally be byref as byval: Result = MyFunction(Arg1, (Arg2)) ' Arg1 is passed byref, arg2 is passed byval

That's confusing enough already. Unfortunately, VB and hence VBScript has some weird rules about when \#3 applies. The rules are

3.1) An argument list for a function call with an assignment to the returned value must be surrounded by parens: Result = MyFunc(MyArg)  
3.2) An argument list for a subroutine call (or a function call with no assignment) that uses the Call keyword must be surrounded by parens: Call MySub(MyArg)  
3.3) If 3.1 and 3.2 do not apply then the list must NOT be surrounded by parens.

And finally there is the byref rule: arguments are passed byref when possible but if there are “extra” parens around a variable then the variable is passed byval, not byref.

Now it should be clear why the statement MySub(MyArg) is legal but MyOtherSub(MyArg1, MyArg2) is not. The first case appears to be a subroutine call with parens around the argument list, but that would violate rule 3.3. Then why is it legal? In fact it is **a subroutine call with no parens around the arg list, but parens around the first argument\!** This passes the argument by value. The second case is a clear violation of rule 3.3, and there is no way to make it legal, so we give an error.

These rules are confusing and silly, as the designers of Visual Basic .NET realized. VB.NET does away with this rule, and insists that all function and subroutine calls be surrounded by parens. This means that in VB.NET, the statement MySub(MyArg) has different semantics than it does in VBScript and VB6 -- this will pass MyArg byref in VB.NET, byval in VBScript/VB6. This was one of those cases where strict backwards compatibility and usability were in conflict, and usability won.

Here's a handy reference guide to what's legal and what isn't in VBScript: Suppose x and y are vars, f is a one-arg procedure and g is a two-arg procedure.

to pass x byref, y byref:  
f x  
call f(x)  
z = f(x)  
g x, y  
call g(x, y)  
z = g(x, y)

to pass x byval, y byref:  
f(x)  
call f((x))  
z = f((x))  
g (x), y  
g ((x)), y  
call g((x), y)  
z = g((x), y)

The following are syntax errors:  
call f x  
z = f x  
g(x, y)  
call g x, y  
z = g x, y

Ah, VBScript. It just wouldn't be the same without these quirky gotchas.

