<div id="page">

# Eval is Evil, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/26/2004 11:13:00 AM

-----

<div id="content">

<span> </span>

<span><span> </span></span>

<div>

<span> </span>

<span></span>

<span>Recall that in [Part One](/ericlippert/archive/2003/11/01/53329.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/01/53329.aspx") we discussed </span><span>eval</span><span> on the client, and in [Part Two](/ericlippert/archive/2003/11/04/53335.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/04/53335.aspx") we discussed </span><span>eval</span><span> on the server.  Today, I answer a question about </span><span>eval</span><span> in JScript .NET. S</span><span>omeone sent me mail the other day asking why this C\# program snippet didn't work: </span>

<span></span>

<span>x = Microsoft.JScript.GlobalObject.eval("1");</span><span> </span>

<span></span>

<span>This throws a seemingly bizarre exception. </span>

<span></span>

<span>Microsoft.JScript.JScriptException: Eval may not be called via an alias </span>

<span></span>

<span>What's up with that?  Well, remember that </span><span>eval</span><span> parses and executes a program snippet **<span>in the context of an existing runtime state</span>**.   </span>

<span></span>

<span>For example, consider this JScript program: </span>

<span></span>

<span>var x = 123;  
</span><span>var y = eval("x + 10"); </span>

<span></span>

<span>How does </span><span>eval</span><span> know to return 133?  </span><span>eval</span><span> **<span>needs to know what the current state of the runtime is</span>** -- what all the variable bindings are, what scope it is presently in, all that kind of stuff.  You can't just up and call a general purpose evaluation engine from no context whatsoever.  Eval isn't a calculator\!  **<span>It's deeply integrated with the JScript .NET runtime. </span>**</span>

<span></span>

<span>If you want to call eval, there are only two legal ways to do so </span>

<span></span>

<span><span>1)<span>     </span></span></span><span>Write a JScript .NET program that calls <span>eval</span><span> </span>explicitly.  
</span><span><span>2)<span>     </span></span></span><span>Construct a </span><span>VsaEngine</span><span> containing the desired runtime state, and then call the public static function </span><span>Eval.JScriptEvaluate(source, engine)</span><span>. </span>

<span></span>

<span>**I don't recommend the second option** unless you *really* know what you are doing. </span>

<span></span>

<span>But what's up with that bizarre exception? Actually, the exception was not designed to catch people trying to call </span><span>eval</span><span> without a runtime context. Though that certainly is a nice side effect, we would have made the error message a little more clear had this been the primary scenario\! </span>

<span></span>

<span>An </span><span>eval</span><span> precludes certain compiler optimizations, so we must detect at compile time whether a given scope contains an </span><span>eval</span><span>. Easy -- just look for calls to that function, right? But you could get around that by aliasing </span><span>eval</span><span>.  Remember, JScript has first class functions, so nothing stops you from saying </span>

<span></span>

<span>function foo(x)  
</span><span>{  
    </span><span>var abc = 123;   
</span><span>    // The compiler detects that only numbers are  
</span><span>    // assigned to abc and optimizes accordingly  
    x("abc = 'hello';");  
</span><span>    // but if x is eval then the optimization is incorrect  
</span><span>    // ...  
</span><span>}  
</span><span>foo(eval)</span><span>;</span><span></span>

<span></span>

<span>We have no way of determining whether or not </span><span>foo</span><span> contains a call to </span><span>eval</span><span>.  **<span>Therefore we made aliasing </span>**</span>**<span>eval</span><span> illegal.</span>**<span>  Now the exception makes sense: </span><span>Eval may not be called via an alias</span><span>. The program above produces that exception. </span>

<span>But why is this exception thrown when a C\# programmer tries this? </span>

<span></span>

<span>x = Microsoft.JScript.GlobalObject.eval("1");</span><span> </span>

<span></span>

<span>Because if the compiler **<span>finds</span>** a call to </span><span>eval</span><span>, it generates a special function call node in the abstract syntax tree which generates different code than the general purpose function call node that would be generated for the case above.  When you say in JScript .NET </span>

<span></span>

<span>eval(whatever); </span>

<span></span>

<span>then the compiler generates a call to </span><span>Eval.JScriptEvaluate(source, engine)</span><span>. After all, the compiler knows where the </span><span>VsaEngine</span><span> state is.  But if you get a reference to </span><span>eval</span><span> via the global object -- either directly in the C\# case, or indirectly in the JScript .NET case -- then you get a dummy function that just throws an exception.</span>

<span>There are also security implications to using <span>eval </span>in JScript .NET which I will get into in a future post.</span><span></span>

</div>

</div>

</div>

