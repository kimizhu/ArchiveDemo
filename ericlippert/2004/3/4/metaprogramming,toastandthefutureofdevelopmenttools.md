# Metaprogramming, Toast and the Future of Development Tools

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/4/2004 11:19:00 AM

-----

Toast\! 

Many, many years ago -- long before they were professors -- Thingo and Orbifold and I were sitting around the Comfy Lounge talking about programming languages for voice-recognition-equipped toasters.  The problem we were considering was **how to talk about activating the toaster without actually activating the toaster**.  This is the analog of the classic language design problem of how to represent quotation marks inside a string -- you want to be able to express the idea "this quotation mark is part of the **content** of the string literal, not the **end** **marker** of this string literal."  

Orbifold figured that the way to do it was the same way VB determines which are the embedded quotes -- any time you wanted to talk **about** the toaster voice protocol, just double it.  To actually **use** the protocol, don't double it.  "To activate my automatic toaster toaster, you just say toaster toaster make make toast toast.  Let me show you:  TOASTER, MAKE TOAST\!"  

Now, that's an easy language parser to write, but it's kind of sub-optimal as far as user experience goes\!  But the goofy sounding grammar was not the fundamental problem I saw in Orbifold's suggestion.  Rather, the problem that I had with his proposal was that the command structure was **imperative**, rather than **declarative**.  I didn't really care how we solved the grammar issue; I wanted the interface to be "I WANT SOME TOAST", not "TOASTER, MAKE TOAST".  I wanted to declare my desires and have the machinery fulfill them, not issue a bunch of commands.  

Fortunately, we have such machinery.  You walk into a restaurant and you can say "I want some toast please", because the toast compilers, I mean wait staff, are smart enough to figure out how to service that request.  You don't have to issue a bunch of commands -- that would be kind of rude actually. 

Now, of course, there really is not much practical difference between "I want toast", and "you: make toast for me\!".  But consider what happens when the abstraction level is lower and the machinery is less automatic.  "I want toast" is a whole lot more efficient than "you: walk into the kitchen, find the bread, insert two slices into the toaster…" , or worse, toaster assembly code for toast-making robots:  "Raise your left leg, shift weight to right leg…" 

This ridiculous-but-true story is actually relevant, I promise.  (Someday I'll tell you the story about the day Thingo, Orbifold and I ate **The Postmodern Gumballs**, which I promise has no relevance to anything.) 

Programs That Write Programs Are The Luckiest Programs In The World 

There is something delightfully magical about programs that write programs.  Excel, for example.  Excel's macro recorder feature writes programs that LOBDevs can then read and modify.  Compilers are also programs that write programs.  The C\# compiler takes in a bunch of strings and writes out a program written in IL.  (Which in turn is consumed by the JIT compiler, which spits out a program written in machine code.) 

We've come a long way in the last few decades.  I was idly flipping through the Dragon Book the other day when I came across this factoid: the [original FORTRAN compiler](http://www.paulgraham.com/history.html "http://www.paulgraham.com/history.html") took **eighteen** man-years to write.  JScript .NET took less than half that, and JScript .NET has *enormously* more expressive power than FORTRAN 1 - but we didn't have to do it in punch cards either. 

And it's not just language wonks like me that benefit from the incredible productivity improvements we've seen in developer tools over the years.  Language wonks are a tiny minority of computer programmers when you consider the number of new line-of-business developers, web developers and other scripters who started programming in the last ten years.  That got me thinking a bit about how programming languages have changed over the years, and how they continue to change.  What features enabled so many non-programmers to become scripters in the 1990's, and what are the ongoing trends in language design? 

I'm going out on a limb here to hazard two predictions:  first, languages will become **more declarative**.  Second, there will be **more special-purpose languages**. 

What do I mean by "imperative" and "declarative"?  I hinted at it a bit above, but let's make it a little more concrete than our ridiculous toaster language. 

function double(x){  
      return x \* 2;  
}  
var y;  
y = 4;  
print(double(y)); 

The green lines don't actually *do* anything, they *describe* the structure of the program.  The program contains a function, the function has a name, takes an argument, has a particular scope, etc.  The blue lines actually *do something* -- move information around, call functions, return control from functions, whatever. The declarative parts describe *what* you want.  The imperative parts describe *how* it works. 

The point I'm getting to, in this somewhat roundabout way, is that declarative language features are often in some sense more abstract, because they don't describe how something works, they describe what you want.  They are therefore much less burden upon the developer, and an increased burden upon the compiler.  Abstraction is increasing.  We are getting better and better at writing smart compilers which can figure out what sequence of actions is required to fulfill a particular declarative request.  This both increases developer productivity and lowers the barrier to entry for developers.  I therefore expect that languages will trend towards more and more declarative features.  

Take HTML for example.  HTML on its own is an almost completely descriptive language.  You don't issue a series of **commands** ("*Move the output cursor to coordinates (10,20).  Output the following text…*"). Rather, you create a **description** of what you want ("*Make me a centered, bold title followed by a form with a button.*..")  In HTML, you describe the page you want and let the display surface take care of turning that into a sequence of commands issued to the operating system.  

This abstraction has great power because it means that your intent is decoupled from the steps needed to achieve it.  The language processor is therefore given free rein to do the right thing in situations that your imperative commands might not have anticipated.  The browser terminal has no advanced graphics capability, or it’s a page reader for the blind, or a cell phone with no mouse, or whatever -- you don't have to write the code that deals with edge cases because the smarts are in the language processor. 

But sometimes you *do* need to say specifically what you want done in what order.  Imperative languages are quite excellent when you require fine-grained low-level control over the specific actions of some subsystem. Declarative languages, particularly markup languages, are really bad at that.  Thus, HTML has special \<SCRIPT\> blocks just for sticking imperative code into the declarative code.  This combination of declarative and imperative code is very powerful, particularly when the declarative and imperative sections can be logically and physically separated.  (I'll discuss the philosophical justifications for this separation in another blog entry; this is getting long enough already.)   

XAML is another example of this trend, which I should also leave for another blog entry.  

This leads me to my second prediction. One can't help but notice that both XAML and HTML are **structured markup languages**.  This trend towards declarative languages will likely be accelerated by the widespread adoption of XML.  Why? Because **XML's easy schematizability makes it easy to create little special-purpose declarative languages that can be automatically syntax checked by schema verifiers**.  

But what about the burden upon the language interpreter?  I suspect that many of the people developing these highly declarative languages will write **translators** that, behind the scenes, are **programs that write programs**.  Ultimately, computers are only good at following instructions, so *something* is going to have to take that declarative description and turn it into imperative code.  But hey, **that's what the CodeDom classes in the .NET Framework are for** -- abstracting code generation.  

That takes care of the language provider's end of the deal, but what about the developer using the language?  Isn't writing XML kind of a pain?  Yes, for humans, but it is a piece of cake for machines.  How many people write raw HTML from scratch?  Most people use FrontPage or some similar tool.   I suspect that in the future **programs that write declarative programs will proliferate**.  A typical day for the pro developer will go something like this.  First, a GUI WYSIWYG designer tool will produce output in the form of a custom, XML-based declarative language.  The developer can then examine that output, tweak it if necessary, and customize it further with imperative code in their imperative language of choice.  The tool then runs the declarative XML through a language-to-codedom translator.  The codedom provider outputs imperative code (again, in the developer's imperative language of choice) implementing the declared semantics.  Then things proceed as they do today -- a compiler compiles the imperative language into IL, and so on.  Abstractions piled on top of abstractions. 

Programs That Write Programs That Write Programs… 

And you know what?  We can accelerate this process by applying these principles to the problem of declarative language creation itself\!  Imagine a program that takes as its input an XSD schema for an XML-based declarative language and an XSLT-like transformation that declaratively describes how elements of that language map onto codedom elements, plus some additional imperative code customizing the hard-to-describe-declaratively bits.  Its output would be** a translator that turns programs written in the XML-based language into any imperative language**.  

That would be a program that writes programs that write programs -- the modern equivalent of YACC.  Perhaps in my incredibly non-existent free time, I'll write such a program.  It certainly would come in handy - believe me, writing language processors can get tedious sometimes\!

