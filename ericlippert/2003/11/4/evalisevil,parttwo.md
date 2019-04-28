# Eval is Evil, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/4/2003 2:23:00 PM

-----

As I promised, more information on why eval is evil.  (We once considered having T-shirts printed up that said "Eval is evil\!" on one side and "Script happens\!" on the other, but the PM's never managed to tear themselves away from their web browsing long enough to order them.)

  

 

Incidentally, a buddy of mine who is one of those senior web developer kinda guys back in Waterloo sent me an email yesterday saying "Hello, my name is Robert and I am an evalaholic".  People, it wasn't my *intention* to start a twelve step program, but hey, whatever works\! 

 

 

As I [discussed the other day](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/5f27ae83-ff82-4fea-97db-b6fef3922c3b), eval on the client is evil because it leads to sloppy, hard-to-debug-and-maintain programs that consume huge amounts of memory and run unnecessarily slowly even when performing simple tasks.  But like I said in my [performance rant](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/bfd7cd56-caf1-47e5-b94a-0c812b1be28e), if it's **good enough**, then hey, it's good enough.  Maybe you don't **need** to write maintainable, efficient code.  Seriously\! Script is often used to write programs that are used a couple of times and then thrown away, so who cares if they're slow and inelegant?

 

 

But eval on the server is an entirely different beast.  First off, server scenarios are generally a lot more performance sensitive than client scenarios.  On a client, once your code runs faster than a human being can notice the lag, there's usually not much point in making it faster.  But  [as I mentioned earlier](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/ae29f2ce-b06d-47a1-9520-857c350e3988), ASP goes to a lot of work to ensure that for a given page, the compiler only runs once. An eval defeats this optimization by making the compiler run every time the page runs\! On a server, going from 25 ms to 40 ms to serve a page means going from 40 pages a second to 25 pages a second, and that can be expensive in real dollar terms.  

 

 

But that's not the most important reason to eschew eval on the server.  Any use of eval (or its VBScript cousins [Eval, Execute and ExecuteGlobal](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/4939ad1e-b2d7-436e-a2dc-bd7665d207bf)) is a **potentially enormous security hole.  ** 

 

 

\<% 

  var Processor\_ProductList;

  var Software\_ProductList;

  var HardDisk\_ProductList;

  // ...

  CategoryName = Request.QueryString("category"); 

  ProductList = eval(CategoryName & "\_ProductList"); 

  // ...

 

 

What's wrong with this picture?  **The server assumes that the client is not hostile.**  Is that a warranted assumption?  Probably not\!  You know nothing about the client that sent the request.  Maybe **your** client page only sends strings like "Processor" and "HardDisk" to the server, but anyone can write their own web page that sends 

 

 

((new ActiveXObject('Scripting.FileSystemObject')).DeleteFile('C:\*.\*',true)); Processor

 

 

which will cause eval to evaluate

 

 

((new ActiveXObject('Scripting.FileSystemObject')).DeleteFile('C:\*.\*',true)); Processor\_ProductList

 

 

Obviously that's a pretty unsophisticated attack.  The attacker can put [any code in there that they want](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/c40acab4-2215-40c6-999c-2ad2d0d05833), and it will run in the context of the server process.  Hopefully the server process is not a highly privileged one, but still, there's vast potential for massive harm here just by screwing up the logic on your server.

 

 

**Never trust the input to a server, and try to never use eval on a server.  Eval injection makes SQL injection look tame\!**

 

 

To try and mitigate these sorts of problems, JScript .NET has some restrictions on its implementation of eval, but that's a topic for another entry.

