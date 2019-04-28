# Asynchrony in C\# 5, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/28/2010 11:30:00 AM

-----

The designers of C\# 2.0 realized that writing iterator logic was painful. So they added **iterator blocks.** That way the compiler could figure out how to build a state machine that could store the continuation - the “what comes next” - in state somewhere, hidden behind the scenes, so that you don’t have to write that code. They also realized that writing little methods that make use of local variables was painful. So they added **anonymous methods**. That way the compiler could figure out how to hoist the locals to a closure class, so that you don't have to write that code. The designers of C\# 3.0 realized that writing code that sorts, filters, joins, groups and summarizes complex data sets was painful. So they added **query comprehensions** and all the rest of the LINQ features. That way the compiler could figure out how to do the right object model calls to build the query, the expression trees, and so on. The designers of C\# 4.0 realized that interoperating with modern and legacy dynamic object models was painful. So they added the **dynamic type.** That way the compiler could figure out how to generate the code at compile time that does the analysis in the Dynamic Language Runtime at runtime. The designers of C\# 5.0 realized that writing asynchronous code is painful, in so many ways. Asynchronous code is hard to reason about, and as we've seen, the transformation into a continuation is complex and leads to code replete with mechanisms that obscure the meaning of the code. **This shall not stand.** I am pleased to announce that there will be a C\# 5.0 (\*), and that in C\# 5.0 you’ll be able to take this synchronous code:  

void ArchiveDocuments(List\<Url\> urls)  
{  
  for(int i = 0; i \< urls.Count; ++i)  
    Archive(Fetch(urls\[i\]));  
}

and, given reasonable implementations of the FetchAsync and ArchiveAsync methods, transform it into this code to achieve the goal of sharing wait times as described yesterday:  

async void ArchiveDocuments(List\<Url\> urls)  
{  
  Task archive = null;  
  for(int i = 0; i \< urls.Count; ++i)  
  {  
    var document = await FetchAsync(urls\[i\]);  
    if (archive \!= null)  
      await archive;  
    archive = ArchiveAsync(document);  
  }  
}

Where is the state machine code, the lambdas, the continuations, the checks to see if the task is already complete? They’re all still there. **Let the compiler generate all that stuff for you,** just like you let the compiler generate code for iterator blocks, closures, expression trees, query comprehensions and dynamic calls. The C\# 5.0 code above is essentially a syntactic sugar for the code I presented yesterday. That's a pretty sweet sugar\! I want to be quite clear on this point: **the action of the code above will be logically the same as the action of yesterday's code**. Whenever a task is "awaited", the remainder of the current method is signed up as a continuation of the task, and then control immediately returns to the caller. When the task completes, the continuation is invoked and the method starts up where it was before. If I’ve timed the posting of this article correctly then Anders is announcing this new language feature at the PDC right about… now. You can watch it [here](http://player.microsoftpdc.com/Session/1b127a7d-300e-4385-af8e-ac747fee677a). We are as of right now making a **Community Technology Preview edition** of the prototype C\# 5.0 compiler available. The prototype compiler will be of prototype quality; expect features to be rough around the edges still. The idea is to give you a chance to try it and see what you think so that we can get early feedback. I'll be posting more about this feature tomorrow and for quite some time after that; if you can't wait, or want to get your hands on the prototype, you can get lots more information and fresh tasty compiler bits from [msdn.com/vstudio/async](http://msdn.com/vstudio/async). And of course, in keeping with our co-evolution strategy, there will be a Visual Basic 11 and it will also feature task-based asynchrony. Check out the [VB team blog for details](http://blogs.msdn.com/b/vbteam/archive/2010/10/28/async.aspx), or [read all about this feature at my colleague Lucian's blog](http://blogs.msdn.com/b/lucian/archive/tags/async/). (Lucian did much of the design and prototyping of this feature for both C\# and VB; he, not I, is the expert on this so if you have deep questions, you might want to ask him, not me.) **Tomorrow**: await? async? Task? what about AsyncThingy\<T\>? Tell me more\!

(\*) We are absolutely positively not announcing any **dates** or **ship vehicles** at this time, so don't even ask. Even if I knew, which I don't, and even if my knowledge had the faintest chance of being accurate, which it doesn't, I still wouldn't tell you.

