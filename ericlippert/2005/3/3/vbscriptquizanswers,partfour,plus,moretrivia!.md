# VBScript Quiz Answers, Part Four, Plus, More Trivia\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/3/2005 10:06:00 AM

-----

Last night I gave away more copies of my book at our Visual Studio Tools for Office developer lab, but this time I made the trivia questions a whole lot easier. I played five rounds of "Odd Man Out" -- you know, like from Sesame Street. One of these things is not like the other, one of these things just doesn't belong… Just for fun, how many of them can you get?  
  
A) Mort, Elvis, Einstein, Roger  
B) WMI, ADO, DAO, ODBC  
C) Gudgeon, Springlering, Pintle, Tumblehome  
D) Frodo, Pippin, Bilbo, Drogo  
E) Chicago, Daytona, Everett, Cairo Spoiler warning: I'll put the answers in the comments to this post. Pizza, beer, VSTO demos and dumb trivia questions -- it's a developer paradise over here, I tell you\! Anyway, moving swiftly on with the answers to my VBScript quiz: 4) Inside a class definition, which of the following are syntactically legal? Why? (a) Private Default Sub Default()  
(b) Public Default Function Default  
(c) Public Default  
(d) Public Default Property Get Default(Property) (a) is illegal, the rest are legal. (a) is illegal because it makes no sense to have a private default method.  The rest are legal because it is perfectly legal to use Default and Property as the names of member functions, properties and variables. Why? We added the Default and Property keywords to the language in VBScript v5. We were afraid that we would break existing programs from a previous version if suddenly they became illegal to use as identifiers. If anyone happened to use Default or Property as a function or variable name, that needed to keep on working.  (And besides, VB6 doesn't reserve Default or Property either.) We did have to make Class a reserved word. If Class is allowed as an identifier then you can get into this situation: Sub Class(x)  
End Sub  
Class Foo  
  
and it becomes very difficult to know whether you are calling the procedure or introducing a new class. Hopefully not very many people were using Class as a variable name, because I broke all of them.

