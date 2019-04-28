# Eval is Evil, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/26/2004 11:13:00 AM

-----

 

 

 

Recall that in [Part One](/ericlippert/archive/2003/11/01/53329.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/01/53329.aspx") we discussed eval on the client, and in [Part Two](/ericlippert/archive/2003/11/04/53335.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/04/53335.aspx") we discussed eval on the server.  Today, I answer a question about eval in JScript .NET. Someone sent me mail the other day asking why this C\# program snippet didn't work: 

x = Microsoft.JScript.GlobalObject.eval("1"); 

This throws a seemingly bizarre exception. 

Microsoft.JScript.JScriptException: Eval may not be called via an alias 

What's up with that?  Well, remember that eval parses and executes a program snippet **in the context of an existing runtime state**.   

For example, consider this JScript program: 

var x = 123;  
var y = eval("x + 10"); 

How does eval know to return 133?  eval **needs to know what the current state of the runtime is** -- what all the variable bindings are, what scope it is presently in, all that kind of stuff.  You can't just up and call a general purpose evaluation engine from no context whatsoever.  Eval isn't a calculator\!  **It's deeply integrated with the JScript .NET runtime. **

If you want to call eval, there are only two legal ways to do so 

1\)     Write a JScript .NET program that calls eval explicitly.  
2\)     Construct a VsaEngine containing the desired runtime state, and then call the public static function Eval.JScriptEvaluate(source, engine). 

**I don't recommend the second option** unless you *really* know what you are doing. 

But what's up with that bizarre exception? Actually, the exception was not designed to catch people trying to call eval without a runtime context. Though that certainly is a nice side effect, we would have made the error message a little more clear had this been the primary scenario\! 

An eval precludes certain compiler optimizations, so we must detect at compile time whether a given scope contains an eval. Easy -- just look for calls to that function, right? But you could get around that by aliasing eval.  Remember, JScript has first class functions, so nothing stops you from saying 

function foo(x)  
{  
    var abc = 123;   
    // The compiler detects that only numbers are  
    // assigned to abc and optimizes accordingly  
    x("abc = 'hello';");  
    // but if x is eval then the optimization is incorrect  
    // ...  
}  
foo(eval);

We have no way of determining whether or not foo contains a call to eval.  **Therefore we made aliasing ****eval illegal.**  Now the exception makes sense: Eval may not be called via an alias. The program above produces that exception. 

But why is this exception thrown when a C\# programmer tries this? 

x = Microsoft.JScript.GlobalObject.eval("1"); 

Because if the compiler **finds** a call to eval, it generates a special function call node in the abstract syntax tree which generates different code than the general purpose function call node that would be generated for the case above.  When you say in JScript .NET 

eval(whatever); 

then the compiler generates a call to Eval.JScriptEvaluate(source, engine). After all, the compiler knows where the VsaEngine state is.  But if you get a reference to eval via the global object -- either directly in the C\# case, or indirectly in the JScript .NET case -- then you get a dummy function that just throws an exception.

There are also security implications to using eval in JScript .NET which I will get into in a future post.

