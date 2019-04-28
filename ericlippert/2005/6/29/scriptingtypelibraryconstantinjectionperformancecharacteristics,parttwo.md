# Scripting Type Library Constant Injection Performance Characteristics, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/29/2005 2:02:00 PM

-----

Script developers can declare constants, variables, functions and classes at global scope by writing the appropriate lines of code. Script *hosts* (Internet Explorer, ASP, WSH, etc) however **can only add objects** to the script engine’s global scope. For practical purposes, it’s as though the host creates a new global variable which is assigned to a particular object and cannot be changed. I suppose that we could have enabled hosts to add named constants pretty easily -- the host could pass in a name and a variant containing the value. We could even do host variables by having the host pass in the address of a variant containing the variable. There's no obvious way to do host-supplied functions other than having the host provide a named object with a default property that calls the function, and now we're back where we started with host-added objects.  It's also kind of a pain in the rear to provide a hundred different function objects if you have a hundred functions to inject. Fortunately we can effectively get host-provided constants, variables and functions even if hosts can only add named objects. How? By enabling the host to pass in flags to say "**the members of this object may/must/cannot be accessed by qualifying with the object name**". That's how Internet Explorer does it. IE adds the window object to the script engine with the flag "you may access members of window without qualification". Therefore, both window.document.whatever() and document.whatever() work. ASP, by contrast, sets the flags indicating that members of the Response object must be accessed via Response.Write. Just plain Write doesn’t work. Given this mechanism, the other mechanisms become unnecessary. If the host wants to add a named constant then the host just creates an object, sets the appropriate flags, and gives the object a property with a getter, no setter. From the scripter's point of view, it’s just like the constant was added to the global namespace. Back to type library importing. We’ve got typelib Cheese {  
  enum Blue{  
    Stilton = 1  
    Gorgonzola = 2  
  }  
} so we want to add two constants to the global namespace, plus two objects, so that we can say foo.bar = Stilton  
foo.bar = Blue.Stilton        ' Blue must be an object  
foo.bar = Cheese.Blue.Stilton ' Cheese must be an object At this point we were faced with two choices. We could come up with some new mechanism whereby constants could be added to the script engine, or we could re-use the existing named object injection mechanism. The former wouldn’t have been that hard, but we chose the latter. When you inject a type library into the engine, what we do is we build up an object with the following properties: Cheese.Stilton  
Cheese.Gorgonzola  
Cheese.Blue where Cheese.Blue is itself an object with the right properties. We then inject Cheese as a named item into the script engine and mark it as "may be qualified". This solution has the (arguably) nice property that you can disambiguate with only the outer name as well. That is, you can say foo.bar = Cheese.Stilton That’s maybe a little weird. We could have instead built the object hierarchy by not putting the bottom-level values on the top-level object, and instead added the top and mid-level objects as named items instead. That is, create Cheese with only property Blue, and Blue with properties Stilton and Gorgonzola, and then add both Cheese and Blue as "may be qualified" named items. That solution presents additional problems though. Suppose we did that, and also added typelib Food {  
  enum Cheese {  
    Green = 1  
    Blue = 2  
    Yellow = 3  
    White = 4  
  }  
} We’d then have two top-level items both named Cheese, both with a Blue property, and it would be ambiguous which was which – we’d need to invent a mechanism for resolving conflicts amongst global named items, which is not something we really wanted to do. So far we’ve seen how type library injection works at a high level. Next time we’ll delve into some of the performance issues that arise.

