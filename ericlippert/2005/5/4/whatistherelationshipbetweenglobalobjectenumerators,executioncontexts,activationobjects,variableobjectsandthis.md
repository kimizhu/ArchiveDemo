# What is the relationship between global object enumerators, execution contexts, activation objects, variable objects and this?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2005 2:50:00 PM

-----

UPDATE: I am WRONG WRONG WRONG. Brendan Eich, the original creator of JavaScript, was kind enough to point out at great length that I was WRONG WRONG WRONG in my conclusions earlier today.  Somehow I managed to miss the key section in the spec -- my search-fu is not all its cracked up to be apparently. How embarrassing\! Allow me to correct the offending conclusion. \*\*\*\*\*\*\*\*\*\*\*\* Here's an interesting question that I received earlier this year from a reader. The reader notes that in the Gecko implementation of ECMAScript you can enumerate the properties of the global object and get back a list of global functions but in JScript you cannot.  \*\*\*\*\*\*\*\*\*\*\*\* As I understand JavaScript, global functions should be properties of global; that is, in the global scope chain. But in JScript I have to explicitly define a function as a member of global "this" to be able to iterate it.  
  
var **global** = this;  
function invisibleToIE() {  
  alert("IE can't see me");  
}  
\_\_global\_\_.visibleToIE = function() {  
  alert("IE sees me");  
}  
  
for (func in **global**) {  
  var f = **global**\[func\];  
  if (func.match(/visible/)) {  
    f();  
  }  
}  
  
Shouldn't  function foo() { }  and  this.foo = function() { } be completely identical? If so, should I then not be able to get "foo" as a property of global this? Do you know of a workaround for this? Is this a bug in JScript? or in Gecko JS? Or is this question up in the air? \*\*\*\*\*\*\*\*\*\*\*\* In fact, **JScript/IE gets it wrong and Gecko gets it right**.  Explaining why will require some background though. Let's start by considering a related scenario in which we define a closure.  function MyObject(){}  
function MakeAdder(first) {  
  function Adder(second) {return first + second; }   
  return Adder;  
}  
MyObject.prototype.MakeAdder = MakeAdder;  
var x = new MyObject();  
var AddTwo = x.MakeAdder(2);  
print(AddTwo(3));  
  
When MakeAdder is called, "this" refers to global variable x.  **Does declaring Adder automatically add an "Adder" property to x? ** Of course not\!  That function is a **variable bound to the local scope**, not a member bound to the "this" object. Then what should this do? function MakeAdder(first) {  
  function Adder(second) {return first + second; }   
  for(var yyy in this)  
     print(yyy);  
  return Adder;  
} Should that print "Adder" as one of the properties of this? Of course not.  As we've just established, Adder is a local variable, not a property of this. Now just remove the outer function, and we're in exactly the same situation, just in the global scope rather than a function scope.  Declaring functions in global scope has nothing whatsoever to do with changing the "this" object of the global scope.  It changes the **variable list** of the global scope, sure, but not the "this" object. **Or does it?** Let's take a walk through the relevant sections of the specification. Section 13 describes the semantics of a function declaration. The production **FunctionDeclaration : function Identifier ( FormalParameterList ) { FunctionBody }** is processed for function declarations as follows:  
1.      Create a new Function object as specified in section 13.2 with parameters specified by FormalParameterList, and body specified by FunctionBody. Pass in the scope chain of the running execution context as the Scope.  
2.      Create a property of the current variable object \[…\] with name Identifier and value Result(1). What is the "running execution context"?  Basically either the global context or a function.  Section 10 says: When control is transferred to ECMAScript executable code, control is entering an execution context. Active execution contexts logically form a stack. The top execution context on this logical stack is the running execution context. What is the "current variable object"?  Section 10.1.3 says: Every execution context has associated with it a variable object. Variables and functions declared in the source text are added as properties of the variable object. \[…\] For each *FunctionDeclaration* in the code, in source text order, create a property of the variable object whose name is the *Identifier* in the *FunctionDeclaration*, whose value is the result returned by creating a Function object as described in section 13, and whose attributes are determined by the type of code. If the variable object already has a property with this name, replace its value and attributes. But what *exactly* is the relationship between an execution context and a variable object?  Section 10.1.6 clears that up: When control enters an execution context for function code, an object called the activation object is created and associated with the execution context. \[…\] The activation object is then used as the variable object for the purposes of variable instantiation. The activation object is purely a specification mechanism. It is impossible for an ECMAScript program to access the activation object. It can access members of the activation object, but not the activation object itself. Now, in our example of a global function, there is a single global activation object, which is the global variable object. Let me just re-emphasize that the variable object is distinct from the "this" object, as noted in section 10.1.7: There is a **this** value associated with every active execution context. The this value depends on the caller and the type of code being executed and is determined when control enters the execution context. The this value associated with an execution context is immutable. Got it?  An execution context contains a "this" object obtained from the caller.  An execution context also contains an activation object which is created fresh every time the context is entered.  The context uses the activation object as its variable object.  New declarations are set as properties on the variable object, not the "this" object -- the "this" object and the activation object are possibly completely different objects.  This explains how it is that new local functions and variables are not bound to the "this" that comes from their caller. The kicker (which I missed in the earlier version of this article) is section 10.2.1: \[Global\] Variable instantiation is performed **using the global object as the variable object** and using property attributes { DontDelete }.  
**The \[global\] this value is the global object.** (My emphasis.) Aha\! For the global context, the global variable object and the global this are the **same object**. Therefore, it is NOT like the function scope example.  Adding a new function to the global variable object should add a new object to the global this. So to answer the specific questions: Shouldn't function foo() { } and  this.foo = function() { }   be completely identical? In a *function scope* there is no requirement that they be the same.  One binds a variable in an execution context, the other sets a property on an object.  Those are completely different.  But in the *global scope*, those two things are the same. Therefore, JScript/IE is broken. Do you know of a workaround for this? I do not know of any way to enumerate global functions from within the language. Behind the scenes the global variable object is actually implemented as a JScript object, and the global this object is implemented by the host.  Therefore JScript they are different objects, and there is no way to peek behind the scenes and get ahold of the variable object itself *within the language*.  However the language *host* is capable of enumerating the global variable object if it wants to.  If you are implementing your own script host, call GetScriptDispatch and that will give you back our internal pointer to the global variable object.  (This is how Internet Explorer implements being able to call JScript functions from a VBScript engine on the same page, but that's another story.) Is this a bug in JScript? or in Gecko JS? Or is this question up in the air? It is a bug in JScript, or IE.  I'm not sure which yet.  I will follow up with the Sustaining Engineering team and see if we can get this fixed in a future version of IE.  However, I wouldn't hold my breath waiting if I were you.  Thanks for the interesting question and valuable feedback.

