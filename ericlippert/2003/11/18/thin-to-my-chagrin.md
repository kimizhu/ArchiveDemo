<div id="page">

# Thin To My Chagrin

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/18/2003 2:29:00 PM

-----

<div id="content">

<span> </span>

<div>

<span>I'm going to take a quick intermission from talking about the type system, but we'll pick it up again soon.  I've been thinking a lot lately about philosophical and practical issues of thin client vs. rich client development.  Thus, I ought to first define what I mean by "thin client" and "rich client".  </span>

<span>Theory </span>

<span></span>

<span>We used to think of Windows application software as being shrink-wrapped boxes containing monolithic applications which maybe worked together, or maybe were "standalone", but once you bought the software, it was static -- it didn't run around the internet grabbing more code.  If the application required buff processing power and lots of support libraries in the operating system, well, you needed to have that stuff on the client.  The Windows application model required that the client have a rich feature set. </span>

<span></span>

<span>This is in marked contrast to the traditional Unix model of application software.  In this model the software lives on the server and a bunch of "thin" clients use the server.  The clients may have only a minimal level of memory and processing power -- perhaps only the ability to display text on a screen\! </span>

<span></span>

<span>The ongoing explosion of massive interconnection via the Internet starting in the 1990's naturally led software developers to rethink the monolithic application model.  Multi-tiered development was the result -- the front end that the user sees is just a thin veneer written in HTML, while the underlying business logic and data storage happens on servers behind the scenes.  The client has to be quite a bit fatter than a Unix "dumb terminal", but if it can run a web browser, it's fat enough.  </span>

<span></span>

<span>This was a great idea in many ways.  Multi-tiered development encourages encapsulation and data locality.  Encapsulating the back end means that you can write multiple front-end clients, and shipping the clients around as HTML means that you can automatically update every client to the latest version of the front end.  My job from 1996 to 2001 was to work on the implementation of what became the primary languages used on the front-end tier (JScript) and the web server tier (VBScript).  It was exciting work. </span>

<span></span>

<span>Right now, we're looking to the future.  We've made a good start at letting people develop thin-client multi-tiered applications in Windows, but there is a lot more we can do.  To do so, we need to understand what exactly is goodness.  So let me declare right now Eric Lippert's Rich Client Manifest </span>

<span></span>

**<span>The thin-client multi-tiered approach to software development squanders the richness available on the vast majority of client platforms that I'm interested in.  We must implement tools that allow rich client application developers to attain the benefits of the thin-client multi-tiered model. </span>**

<span></span>

<span>That's the whole point of the .NET runtime and the coming Longhorn API.  The thin client model lets you easily update the client and keeps the business logic on the back tier?  Great -- let's do the same thing in the rich client world, so that developers who want to develop front ends that are more than thin HTML-and-script shells can do so without losing the advantages that HTML-and-script afford.  </span>

<span> </span>

<div>

<span></span>

</div>

<span></span>

<span><span>Practice</span></span>

<span>I've been thinking about this highfalutin theoretical stuff recently because of some eminently practical concerns.  Many times over the years I've had to help out third party developers who have gotten themselves into the worst of both worlds.  A surprisingly large number of people look at the benefits of the thin client model -- easy updates (the web), a declarative UI language (HTML), an easy-to-learn and powerful language (JScript) -- and **<span>decide that this is the ideal environment to develop a rich client application</span>**. </span>

<span></span>

<span>That's a bad idea on so many levels.  Remember, it is called the **<span>thin</span>** client model for a reason.  I've seen people who tried to develop *<span>database management systems</span>* in JScript and HTML\!  That's a thorough abuse of the thin client model -- in the thin client model, the database logic is done on the backend by a dedicated server that can handle it, written by database professionals in hand-tuned C.  **<span>JScript was designed for simple scripts on simple web pages, not large-scale software. </span>**</span>

<span></span>

<span>Suppose you were going to design a language for thin client development and a language for rich client development.  What kinds of features would you want to have in each? </span>

<span></span>

<span>For the thin client, you'd want a language that had a very simple, straightforward, learn-as-you-go syntax.  The **<span>concept count</span>**, the number of concepts you need to understand before you start programming, should be low.  The "hello world" program should be something like </span>

<span></span>

<span>print "hello, world\!" </span>

<span></span>

<span>and not </span>

<span></span>

<span>import library System.Output;  
</span><span>public startup class MainClass  
</span><span>{   
</span><span>  public static startup function Main () : void   
</span><span>  {   
</span><span>     System.Output("hello, world\!");   
</span><span>  }  
</span><span>}; </span>

<span></span>

<span>It should allow novice developers to easily use concepts like variables and functions and loops.  It should have a loose type system that coerces variables to the right types as necessary.  It should be garbage collected.  There must not be a separate compile-and-link step.  The language should support late binding.  The language will typically be used for user interface programming, so it should support event driven programming.  High performance is unimportant -- as long as the page doesn't appear to hang, its fast enough.  It should be very easy to put stuff in global state and access it from all over the program -- since the program will likely be small, the lack of locality is relatively unimportant.  </span>

<span></span>

<span>In short, the language should enable **<span>rapid development of simple software by relatively unsophisticated programmers through a flexible and dynamic programming model.</span>**  </span>

<span></span>

<span>OK, what about the rich-client language?  The language requirements of large-scale software are **<span>completely different</span>**.  The language must have a rigid type system that catches as many problems as possible before the code is checked in.  There must be a compilation step, so that there is some stage at which you can check for warnings.  It must support modularization, encapsulation, information hiding, abstraction and re-use, so that large teams can work on various interacting components without partying on each other's implementation details.  The state of the program may involve manipulating scarce and expensive resources -- large amounts of memory, kernel objects such as file handles, etc.  Thus the language should allow for fine-grained control over the lifetime of every byte. </span>

<span></span>

<span>Object Oriented Programming in C++ is one language and style that fits this bill, but the concept count of C++ OOP is enormous -- pure, virtual, abstract, instance, static, base, pointers, references...  That means that you need sophisticated, highly educated developers.  The processing tasks may be considerable, which means that performance becomes a factor.  Having a complex "hello world" is irrelevant, because no one uses languages like this to write simple programs. </span>

<span></span>

<span>In short, a rich-client language should support **<span>large-scale development of complex software by large teams of sophisticated professional programmers through a rigid and statically analyzable programming model. </span>**</span>

<span></span>

<span>Complete opposites\!  Now, what happens when you try to write a rich client style application using the thin client model?  </span>

<span></span>

*<span>Apparent</span>*<span> progress will be **<span>extremely rapid</span>** -- we designed JScript for rapid development.  Unfortunately, **<span>this</span>** **<span>rapid development masks serious problems</span>** festering beneath the surface of *<span>apparently</span>* working code, problems which will not become apparent until the code is an unperformant mass of bugs.  </span>

<span></span>

<span>Rich client languages like C\# force you into a discipline -- the standard way to develop in C\# is to declare a bunch of classes, decide what their public interfaces shall be, describe their interactions, and implement the private, encapsulated, abstracted implementation details.  That discipline is required if you want your large-scale software to not devolve into an undebuggable mess of global state.  If you can modularize a program, you can design, implement and test it in independent parts. </span>

<span></span>

<span>It is possible to do that in JScript, but the language does not by its very nature lead you to do so.  Rather, it leads you to to favour expedient solutions (call </span><span>eval</span><span>\!) over well-engineered solutions (use an optimized lookup table).  Everything about JScript was designed to be as dynamic as possible. </span>

<span></span>

<span>Performance is particularly thorny.  Traditional rich-client languages are designed for speed and rigidity.  JScript was designed for comfort and flexibility.  JScript is not fast, and it uses a lot of memory.  Its garbage collector is optimized for hundreds, maybe thousands of outstanding items, not hundreds of thousands or millions. </span>

<span></span>

<span>So what do you do if you're in the unfortunate position of having a rich client application written in a thin-client language, and you're running into these issues? </span>

<span></span>

<span>It's not a good position to be in. </span>

<span></span>

<span>Fixing performance problems after the fact is extremely difficult.  The way to write performant software is to first decide what your performance goals are, and then to MEASURE, MEASURE, MEASURE all the time.  Performance problems on bloated thin clients are usually a result of what I call "frog boiling".  You throw a frog into a pot of boiling water, it jumps out.  You throw a frog into a pot of cold water and heat it up slowly, and you get frog soup.  That's what happens -- it starts off fine when it is a fifty line prototype, and every day it gets slower and slower and slower... if you don't measure it every day, you don't know how bad its getting until it is too late.  The best way to fix performance problems is to never have them in the first place. </span>

<span></span>

<span>Assuming that you're stuck with it now and you want to make it more usable, what can you do? </span>

<span></span>

  - **<span>Data is bad</span>**<span>. Manipulate as little data as possible.  That's what the data tier is for.  If you must manipulate data, keep it simple -- use the most basic data structures you can come up with that do the job. </span>

<span></span>

  - **<span>Code is worse</span>**<span>.  Every time you call </span><span>eval</span><span>, performance sucks a little bit more.  Use lookup tables instead of calling </span><span>eval</span><span>.  Move code onto the server tier.  </span>

<span></span>

  - **<span>Avoid closures</span>**<span>.  Don't nest your functions unless you **<span>really understand</span>** closure semantics and **<span>need</span>** them. </span>

<span></span>

  - **<span>Do not rely on "tips and tricks" for performance</span>**<span>.  People will tell you "*<span>declared variables are faster than undeclared variables</span>*" and "*<span>modulus is slower than bit shift</span>*" and all kinds of nonsense.  Ignore them.  That's like mowing your lawn by going after random blades of grass with nail scissors.  You need to find the WORST thing, and fix it first.  That means measuring.  Get some tools -- Visual Studio Analyzer can do some limited script profiling, as can the Numega script profiler, but even just putting some logging into the code that dumps out millisecond timings is a good way to start.  Once you know what the slowest thing is, you can concentrate on modularizing and fixing it. </span>

<span></span>

  - **<span>Modularize</span>**<span>.  Refactor the code into clean modules with a well-defined interface contract, and test modules independently.  </span>

<span></span>

<span>But the best advice I can give you is simply **<span>use the right tool for the right job</span>**.  The script languages are fabulous tools for their intended purpose.  So are C\# and C++.  But they really are quite awful at doing each other's jobs\! </span>

</div>

</div>

</div>

