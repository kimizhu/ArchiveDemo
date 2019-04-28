<div id="page">

# T4: VBScript and the Terminator

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/22/2004 3:42:00 PM

-----

<div id="content">

I had some free time on the flight to Canada and my laptop with me, so maybe I will do a little blogging on my vacation after all. A week ago or so a reader asked me to talk a bit about how class termination works in VBScript.  Let’s start by looking at a simple case: Class Foo  
   Public Name  
   Public Other    
   Private Sub Class\_Terminate  
       MsgBox Name & ": Goodbye, world\!"  
   End Sub  
End Class As you can see, the class has an “event handler” which is executed when an instance is just about to be released.  (I put scare quotes on there because, though it has the *form* of an event handler, in fact the underlying mechanism that runs the terminator is completely distinct from the other event handling code in VBScript.) [JScript, as you may know, has a nondeterministic mark-and-sweep garbage collector](http://blogs.msdn.com/ericlippert/archive/2003/09/17/53038.aspx).  It’s “nondeterministic” because it runs pretty much when it feels like it. There is a scheduling algorithm, but from the perspective of the program, it’s pretty much random; it’s quite tricky to predict when it’s going to run.  And it’s “mark and sweep” because every collection, it runs through every script-allocated memory block, checks if it is still accessible by the script, and if it isn’t, the memory is freed.  This makes it mostly immune to the “circular reference” problem, where two or more otherwise dead objects reference each other. VBScript’s garbage collector is completely different.  It runs at the end of every statement and procedure, and does not do a search of all memory.  Rather, it keeps track of everything allocated in the statement or procedure; if anything has gone out of scope, it frees it immediately.  We can see this in action by watching terminators run. Let’s consider a simple example. Sub Bar  
   Dim X  
   Set X = New Foo  
   X.Name = "X"  
End Sub  
MsgBox "Starting Bar"  
Bar  
MsgBox "Done Bar"  
   
outputs, as you’d expect: Starting Bar  
X: Goodbye, world\!  
Done Bar As soon as Bar’s End Sub runs, X goes out of scope.  Since that’s its last reference, the terminator runs. It’s that “last reference” part that’s the rub.  Objects are reference counted in COM – every object keeps track of how many other objects are keeping it alive, and when that drops to zero, the object deletes itself. The trouble is, what if two objects are each keeping a ref on each other, but are otherwise dead?  The “circular reference” problem wasn’t so much a problem before we added classes to VBScript. After all, without the ability to define new objects in script, the only objects you really need to worry about are external ActiveX objects – and not even JScript can break circular references involving ActiveX objects, because the ActiveX objects know nothing about JScript’s garbage collector, and vice versa. But with objects in VBScript, suddenly it becomes quite bad.  You can very easily write scripts that consume memory in the form of objects that reference each other circularly, and they won’t go away when they go out of scope: Sub XYZ  
   Dim Alpha, Bravo  
   Set Alpha = New Foo  
   Alpha.Name = "Alpha"  
   Set Bravo = New Foo  
   Bravo.Name = "Bravo"  
   Set Alpha.Other = Bravo  
   Set Bravo.Other = Alpha  
End Sub MsgBox "Starting XYZ"  
XYZ  
MsgBox "Done XYZ" outputs Starting XYZ  
Done XYZ Uh oh.  Neither of those guys was terminated when their variables went out of scope.  All the memory associated with them is still reserved by the heap. But they’re not reachable now.  There are no variables that reference them.  There’s nothing the script programmer can do about it now. It would be unfortunate indeed if such circular references really did live until the end of the process, when the whole heap is cleaned up, both because we’d leak memory, and because those terminators might be doing something interesting. Fortunately, we can do slightly better than that. If you run the code above in Windows Script Host, you’ll see that the terminators run as soon as the script ends; they are cleaned up eventually.  In IE, the terminators run when the page is torn down.  And in ASP, the terminators run when the page is served.  (Though of course in ASP you don’t want to be putting up message boxes, or accessing the ASP object model – the page is just about to get served, don’t try to mess with it\!) How was this implemented?  And how does this solve the memory leak problem? It’s pretty straightforward. Every time a new instance of a VBScript class is created, it adds itself to a special list maintained by the engine, and whenever one is destroyed, it removes itself from the list.  When the VBScript engine itself is closed by the host, the engine runs down the list and runs all the terminators of all the VBScript objects left on the list. Then it runs down the list a second time and clears every field of the object.  This breaks the circular reference. Better late than never. In Windows Script Host, this really isn’t much help because the engine isn’t closed until the process is about to be torn down anyway. But in IE and even more importantly, ASP, script engines are closed much more aggressively. It was very important that we not add a new feature to the language that made it really easy to write ASP pages which leaked memory and forced you to reboot your server frequently. There are some wrinkles that we ran into when implementing the shutdown logic. Let’s see if you can deduce what they are; I’ll give the answers in my next post. QUESTION \#1: When I first wrote the VBScript class code, that’s not exactly the shutdown sequence I wrote. The original logic ran down the list of objects once, going terminate, clear, terminate, clear, terminate, clear.  Soon after that I rewrote the shutdown sequence to the present terminate, terminate, terminate, clear, clear, clear.  Why did I make the change to a two-pass implementation? What was wrong with the original one-pass implementation? QUESTION \#2: The code that implements VBScript classes checks to see if the instance has been terminated already before it calls the Class\_Terminate method.  It should be clear from the foregoing why that is – if an object is in a circular reference and manages to survive until the engine shuts down, then the object will probably be terminated while there are still references held on it.  The later “clear all fields” phase of the shutdown sequence could then release all the references, triggering the “we’ve just released the last reference, so call the terminator” logic.  We therefore have to protect against the terminator running twice. There’s another way that an object can be terminated twice that we protect against.  In this scenario, not only can the object be terminated twice, but an incorrect implementation of the object deletion code can crash the process.  What is it? Hint: the scenario does not involve circular references. It has just a single object being terminated because it is going out of scope and has no other references. QUESTION \#3: Speaking of which, why do we want to ensure that terminators don’t run twice? QUESTION \#4: This is not a question about termination per se, but there is a termination angle in the answer.  Why do we run the garbage collector at the end of every statement, instead of only at the end of every procedure/global block?  After all, no local variable is going to go out of scope until the end of its procedure\! Doesn’t that seem like a lot of work for no possible gain? "I'll be back" in the new year -- have a good festive holiday season everyone\!  
 

</div>

</div>

