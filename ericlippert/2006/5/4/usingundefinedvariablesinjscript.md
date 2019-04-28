# Using undefined variables in JScript

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2006 12:28:00 PM

-----

I got a question the other day pointing out that in JScript, it is legal to assign a value to an undeclared variable, and the variable kind of gets implicitly declared for you, but it is not legal to read the value of an undeclared variable. The writer wanted to know if y = 1; for some undeclared variable has exactly the same semantics as var y = 1;.

Close, but not quite. Assigning to an undefined variable is *not* equivalent to replacing the assignment with a declaration because *that might declare a local variable bound to a particular activation*. Using a variable without declaring it in Jscript produces a new *global* variable, not a new *local* variable. You can clearly see that they are not equivalent because these two programs do different things:

 

function foo() {y = 1; }  
foo();  
print(y); // 1  
  
  
function foo() {var y = 1; }  
foo();  
print(y); // error, y undefined  

These behaviours of the Jscript engine are conformant to the ECMAScript specification, Revision 3, Section 8.7.

In VBScript, on the other hand, using an undefined variable does produce a new local variable, whether you're reading or writing.

The writer then went on to ask "how can I *prevent* JScript from creating a new variable in this case? I want this to be an error."

To explain, we should take a look at what exactly happens when we see y = 1; so that we can determine that we need to create a new variable. To do the assignment successfully, three things must happen: we must determine an address for y, we must determine the value, and we must store the value in that address. The way those things happen goes like this:

  - First we search the scope chain – with blocks, the activation frame, the globals – for any object that has a property y. We do not find one.
  - Next we search all the global host objects asking them if they have a property y. We do not find one.
  - Therefore we create a special "fake" address and put it on the script stack. This completes the first part of the binding.
  - Determining the value is trivial in this case.
  - When we do the store we detect that there's a fake address on the stack, allocate a new global variable in the script engine's global name table, and replace the fake address with this new real address.
  - Finally, we invoke the property put logic in the global name table to store the value to the correct address.

Notice that the only question we ask the host is "do you have this property?", and if the host says no, we create a variable. Therefore, if you want this to be an error, you have to write a script host that lies to the script engine. If you always say "yes, I have that property" even when you don't, then you can raise an exception when the store logic attempts to do the store. However, I've never heard of anyone trying that. There might be other negative consequences to consistently lying to the script engine about whether you have a property or not. You'll have to try it and see.

As an aside, you might be wondering why we go to the trouble of creating a fake address during the address lookup that gets turned into a real address during the store. Why not just create the real address in the first place?

Ponder that for a moment. Under what circumstances would we not want to create a real global variable until after the right hand side of the assignment was evaluated?

Well, you would not expect this program to produce a new global variable:

 

function foo() { try { y = eval("throw 1;"); } catch(e) { } }  
foo();  
print(y); // should be undefined\!  

The global variable creation must happen at the moment of *actual assignment*, not at the moment when the address is determined, because determining the value might prevent the store from ever happening.

