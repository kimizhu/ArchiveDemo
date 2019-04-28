# Translating intentions and mechanisms

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/25/2008 1:26:00 PM

-----

Before I get into today's blogging, a quick note about my recent post on [How To Not Get A Question Answered](http://blogs.msdn.com/ericlippert/archive/2008/02/20/how-to-not-get-a-question-answered.aspx). That was certainly not *intended* to be fishing for compliments or chiding people for never acknowledging help ten years ago; that said, I appreciate both. Thanks to everyone who made thoughtful comments.

Moving on; a theme that seems to be coming up over and over again in my recent technical conversations is that of *intention* vs. *mechanism*. We have tried hard in C\# 3.0 to make a language where there is a good balance between the code reading as a *declaration* of what *meaning* you intend the code to represent, and reading as a list of *imperative instructions* specifying a *mechanism* which achieves those intentions. I've been accumulating anecdotes about this tension between representing intentions vs. mechanisms; expect this to be a recurring theme in the blog for the next while.

Here's a question I got recently which speaks to this tension with regards to the subject of porting code from one language to another:

> I have some C++ code with a macro in it. A typical usage looks like this: TRY\_WAIT\_OP(Execute()); This macro expands to code similar to this: for(int i=0; i\<10; i++) {  
>     if (Execute()) break;  
>     Sleep(i\*1000);  
> }
> 
> We are translating this code into C\#, but C\# does not have a \#define directive. **How do I do textual replacement of code in C\#?**

Once more, we have someone [looking for a thin metal ruler](http://blogs.msdn.com/ericlippert/archive/2003/11/03/53333.aspx). There's a problem -- represent the meaning of a common operation in a programming language. C++ provides a solution mechanism -- using preprocessor-based metaprogramming to create a nonstandard control flow primitive. The natural tendency when doing a translation is to find the identical mechanism in the new language, and then translate the code to use that mechanism. But what if there is no such mechanism?

In that case, you've got to translate the *intentions*, which after all, is what you are trying to translate in the first place. Presumably the new code is intended to do the same thing as the old code. Translating the mechanisms is just a particularly easy road to translating the intentions. What is the *meaning* of this macro?

Clearly TRY\_WAIT\_OP means "execute some arbitrary code that returns a Boolean. If it returns true, you're done. If it returns false, wait some amount of time and try again, up to ten times".

Now **think about how you would write code *from scratch* that implemented those intentions in C\#.** Don't think at all about how it was written in C++. Your goal here is to solve the same problem using a different tool, so don't use the same techniques that you used for the other tool if they're not appropriate. Use the techniques that are appropriate for *this* tool. The way I would write that in C\# is to write a *method* that takes as its argument *some arbitrary code that returns a Boolean*. "Arbitrary code" is represented in C\# by a *delegate*. We can do a bit better than the macro while we're at it, and return a success code: 

private static bool AttemptMultiple(Func\<bool\> action) {  
    Debug.Assert(action \!= null);  
    const int maxAttempts = 10;  
    const int delay = 1000;  
    for (int attempt = 0 ; attempt \< maxAttempts ; ++attempt) {  
        if (action()) return true;  
        Sleep(attempt \* delay);  
    }  
    return false;  
}

Lambda expression syntax gives us a nice way to do the call:  

AttemptMultiple(()=\>Execute());

A code porting project is a good opportunity to review the design fundamentals:

  - Why 10 retrys?  Should this be a parameter to AttemptMultiple?
  - Why wait 1000 milliseconds instead of some other delay? Should this also be a parameter?
  - Why is the increasing delay linear rather than constant, geometric, etc? Should the delay strategy be a parameter?

The above questions are good but they rather miss the point. The important question is: **is this functionality even a good idea in the first place?** This last point is key. The "try it, fail, wait, try again" strategy is in general a dangerous one because **it does not compose well with itself**. Consider the following:  

bool SendPackets()   { ... if (\!AttemptMultiple(()=\>{ ... })) return false; ... }  
bool TalkToSocket()  { ... if (\!AttemptMultiple(()=\>SendPackets())) return false; ... }  
bool SendData()      { ... if (\!AttemptMultiple(()=\>TalkToSocket())) return false;... }  
void HandleCommand() { ... if (command == SendData && \!AttemptMultiple(()=\>SendData())) ReportErrorToUser();... }

Now suppose that the user has a bad network card and SendPackets is always going to fail. If you look at any *one* of those lines, it looks like the attempt is being made ten times and will take a maximum of about one minute. In fact, the attempt to send the packet is made **ten thousand** times and will not report the error to the user for about a week. Usually the right thing to do when something fails is to go into a failure state *immediately*. **Tell the user that something failed and let them decide when and if to retry it.**

What are some examples of a poor mismatch between intentions and mechanisms that you guys have seen? I'm interested in stories about:

  - mechanisms that subtly did not implement the intentions of the programmer
  - situations where the intention of the code was completely obscured by the mechanisms. How did you make the code better reflect the intentions?
  - situations where the mechanism of the code was important, but obscured by unnecessary emphasis on representing the intention. What were the negative consequences of obscuring the mechanism behind some abstraction?

Thanks\!

