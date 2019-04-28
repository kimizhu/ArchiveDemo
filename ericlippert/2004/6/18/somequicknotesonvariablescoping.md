# Some Quick Notes On Variable Scoping

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/18/2004 9:44:00 AM

-----

Here's a question I got a while back:

> In a VBS or ASP file, the following code doesn't work, which I understand:

Option Explicit  
s = "hello" However, the following code works fine: Option Explicit  
s = "hello"  
Dim s Why can we use a variable before declaring it in VBScript? A similar situation exists in JScript. This is illegal: print(s); But this is legal: print(s);  
var s; What's up with that? Well, let me ask you this -- if you think that looks weird, why do you think this looks normal? Dim s  
s = Foo(123)  
Function Foo(x)  
  Foo = x + 345  
End Function There the function is being used before it is declared, but that doesn't bug you, right? Similarly, variables can be used before they are declared. The behaviour is by design. **Variable declarations and functions are logically "hoisted" to the top of their scope in both VBScript and JScript.** In JScript, both functions and variables may be redeclared at any time. In VBScript, declaring a variable twice in the same script block is illegal, but redefinition in another block is legal. Procedures may be redeclared at will except if the procedure is in a class, in which case redeclaration is illegal. OK, that stuff is reasonably common knowledge, but it gets weirder. Did you know that this is legal in VBScript? s = Foo(123)  
If Blah Then  
  Function Foo(x)  
    Foo = x + 345  
  End Function  
End If Not recommended, but legal.  There's a sad story about why that's legal which I might tell at another time.  Suffice to say that it involves ASP pages, a bug, and a rather recalcitrant online news service.  (This behaves as though the function were declared outside the conditional -- you can't do conditional function definition in VBScript, sorry.) There was a long internal debate over these variable declaration issues back in 1996. There was an even longer debate in the ECMA committee over exactly what the hoisting algorithm should be in ECMAScript. I once knew every such argument about program language syntax in all the scripting languages of men, elves and dwarves -- even now, a score of them come to mind\! But it has been eight long years since 1996. I have seen many battles and many fruitless victories. In short, recalling the details to mind is difficult; it's all pretty much a blur now.  I'm going to have to rely on the spec. In JScript, the hoisting spec is as follows (Section 12.2 of ECMA specification 262, Revision 3)

> If the variable statement occurs inside a **FunctionDeclaration**, the variables are defined with function-local scope in that function \[…\] Otherwise, they are defined with global scope \[…\] Variables are created when the execution scope is entered. A **Block** does not define a new execution scope. Only **Program** and **FunctionDeclaration** produce a new scope. Variables are initialised to **undefined** when created. A variable with an **Initialiser** is assigned the value of its **AssignmentExpression** when the **VariableStatement** is executed, not when the variable is created.

Note that this implies that this silly program print(v);  
{  
  var v = 123;  
}  
print(v); is semantically exactly the same as var v;  
print(v);  
v = 123;  
print(v); Waldemar Horwat's [proposal for ECMAScript 4](http://www.mozilla.org/js/language/es4/core/definitions.html#hoist) further complicates the hoisting algorithm. In E3 there are only **global** and **function** scopes, and declarations are hoisted to the top of the “nearest“ scope. In the proposed E4 spect there are **global**, **package**, **class**, **function**, **block**, **for-statement** and **catch-clause** scopes. In general, declarations are hoisted to the top of their scope, but for backwards compatibility, sometimes variable declarations have to be hoisted to the nearest **global**, **package**, **class** or **function** scope. I want to talk some more about the lack of C++-style block scopes in JScript, but that will have to wait for another day.

