# Restating the problem

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/13/2009 10:36:00 AM

-----

A problem statement:

> I am trying to loop though a sequence of strings. How can I determine when I am on the *last* item in a sequence? I don’t see how to do it with “foreach”.

Indeed, “foreach” does not make it easy to know when you are almost done. Now, if I were foolish enough to actually answer the question as stated, I’d probably say something like this:

You can do so by eschewing “foreach” and rolling your own loop code that talks directly to the enumerator:

 

IEnumerable\<string\> items = GetStrings();  
IEnumerator\<string\> enumtor = items.GetEnumerator();  
if (enumtor.MoveNext())  
{  
  string current = enumtor.Current;  
  while(true)  
  {  
    if (enumtor.MoveNext())  
    {  
      DoSomething(current); // current is not the last item in the sequence.   
      current = enumtor.Current;  
    }  
    else  
    {  
      DoSomethingElse(current); // current is the last item in the sequence.  
      break;  
    }  
  }  
}  
else  
{  
  // handle case where sequence was empty  
}  
  

Yuck. This is *horrid*. A fairly deep nesting level, some duplicated code, the mechanisms of iteration overwhelm any semantic meaning in the code, and it all makes my eyes hurt.

When faced with a question like that, rather than writing this horrid code I’ll usually push back and ask “why do you want to know?”

> I am trying to walk though a sequence of strings and build a string like "a,b,c,d".  After each item I want to place a comma except not after the last item. So I need to know when I’m on the last item.

**Well knowing that certainly makes the problem a whole lot easier to solve, doesn’t it?** A whole bunch of techniques come to mind when given the real problem to solve:

First technique: **find an off-the-shelf part that does what you want.**

In this case, call the ToArray extension method on the sequence and then pass the whole thing to String.Join and you’re done.

Second technique: **Do more work than you need, and then undo some of it.**

Using a string builder, to build up the result, put a comma after every item. Then “back up” when you’re done and remove the last comma (if there were any items in the sequence, of course).

Third technique: **re-state the problem and see if that makes it any easier**.

Consider this equivalent statement of the problem:

> I am trying to walk though a sequence of strings and build a string like "a,b,c,d". *Before* each item I want to place a comma except not before the *first* item. So I need to know when I’m on the first item.

And suddenly the problem is much, much simpler. It’s easy to tell if you’re on the first item in a sequence by just setting a “I’m at the first item” flag outside the foreach loop and clearing it after going through the loop.

When I have a coding problem to solve and it looks like I’m about to write a bunch of really gross code to do so, it’s helpful to take a step back and ask myself “Could I state the problem in an equivalent but different way?” I’ll try to state a problem that I’ve been thinking about iteratively in a recursive form, or vice versa. Or I’ll try to find formalizations that express the problem in mathematical terms. (For example, the method type inference algorithm can be characterized as a graph theory problem where type parameters are nodes and dependency relationships are edges.) Often by doing so I find that the new statement of the problem corresponds more directly to clear code.

