# Closing over the loop variable considered harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/12/2009 6:50:00 AM

-----

(This is part one of a two-part series on the loop-variable-closure problem. [Part two is here](http://blogs.msdn.com/b/ericlippert/archive/2009/11/16/closing-over-the-loop-variable-part-two.aspx).)

-----

**UPDATE**: We **are** taking the breaking change. In C\# 5, the loop variable of a foreach will be logically inside the loop, and therefore **closures will close over a fresh copy of the variable each time**. The "for" loop will not be changed. We return you now to our original article.

-----

I don't know why I haven't blogged about this one before; this is the single most common incorrect bug report we get. That is, someone thinks they have found a bug in the compiler, but in fact the compiler is correct and their code is wrong. That's a terrible situation for everyone; we very much wish to design a language which does not have "gotcha" features like this.

But I'm getting ahead of myself. What's the output of this fragment?

 

var values = new List\<int\>() { 100, 110, 120 };  
var funcs = new List\<Func\<int\>\>();  
foreach(var v in values)  
  funcs.Add( ()=\>v );  
foreach(var f in funcs)  
  Console.WriteLine(f());

Most people expect it to be 100 / 110 / 120.  It is in fact 120 / 120 / 120. Why?

Because ()=\>v means "return **the current value of variable v**", not "return the value v was back when the delegate was created". **Closures close over variables, not over values.** And when the methods run, clearly the last value that was assigned to v was 120, so it still has that value.

This is very confusing. The correct way to write the code is:

 

foreach(var v in values)  
{  
  var v2 = v;  
  funcs.Add( ()=\>v2 );  
}

Now what happens? Every time we re-start the loop body, we logically create a fresh new variable v2. Each closure is closed over a different v2, which is only assigned to once, so it always keeps the correct value.

Basically, the problem arises because we specify that the foreach loop is a syntactic sugar for

 

  {  
    IEnumerator\<int\> e = ((IEnumerable\<int\>)values).GetEnumerator();  
    try  
    {  
      int m; // OUTSIDE THE ACTUAL LOOP  
      while(e.MoveNext())  
      {  
        m = (int)(int)e.Current;  
        funcs.Add(()=\>m);  
      }  
    }  
    finally  
    {  
      if (e \!= null) ((IDisposable)e).Dispose();  
    }  
  }

If we specified that the expansion was

 

    try  
    {  
      while(e.MoveNext())  
      {  
        int m; // INSIDE  
        m = (int)(int)e.Current;  
        funcs.Add(()=\>m);  
      }

then the code would behave as expected.

It's compelling to consider fixing this for a hypothetical future version of C\#, and I'd like to hear your feedback on whether we should do so or not. The reasons FOR making the change are clear; this is a big confusing "gotcha" that real people constantly run into, and LINQ, unfortunately, only makes it worse, because it is likely to increase the number of times a customer is going to use a closure in a loop. Also, it seems reasonable that the user of the foreach loop might think of there being a "fresh" loop variable every time, not just a fresh value in the same old variable. Since the foreach loop variable is not mutable by user code, this reinforces the idea that it is a succession of values, one per loop iteration, and not "really" the same variable over and over again. And finally, the change has no effect whatsoever on non-closure semantics. (In fact, in C\# 1 the spec was not clear about whether the loop variable went inside or outside, since in a world without closures, it makes no difference.)

But that said, there are some very good reasons for not making this change.

The first reason is that obviously this would be a breaking change, and we hates them, my precious. Any developers who depend on this feature, who require the closed-over variable to contain the last value of the loop variable, would be broken. I can only hope that the number of such people is vanishingly small; this is a strange thing to depend on. Most of the time, people do not expect or depend on this behaviour.

Second, it makes the foreach syntax lexically inconsistent. Consider foreach(int x in M()) The header of the loop has two parts, a declaration int x and a collection expression, M(). The int x is to the left of the M(). Clearly the M() is not inside the body of the loop; that thing only executes once, before the loop starts. So *why should something to the collection expression's left be inside the loop*? This seems inconsistent with our general rule that stuff to the left logically "happens before" stuff to the right. The declaration is lexically NOT in the body of the loop, so why should we treat it as though it were?

Third, it would make the "foreach" semantics inconsistent with "for" semantics. We have this same problem in "for" blocks, but "for" blocks are much looser about what "the loop variable" is; there can be more than one variable declared in the for loop header, it can be incremented in odd ways, and it seems implausible that people would consider each iteration of the "for" loop to contain a fresh crop of variables. When you say for(int i; i \< 10; i += 1) it seems dead obvious that the "i += 1" means "increment the loop variable" and that there is one loop variable for the whole loop, not a new fresh variable "i" every time through\! We certainly would not make this proposed change apply to "for" loops.

And fourth, though this is a nasty gotcha, there is an easy workaround, and tools like ReSharper detect this pattern and suggest how to fix it. We could take a page from that playbook and simply issue a compiler warning on this pattern. (Though adding new warnings brings up a whole raft of issues of its own, which I might get into in another post.) Though this is vexing, it really doesn't bite that many people that hard, and it's not a big deal to fix, so why go to the trouble and expense of taking a breaking change for something with an easy fix?

Design is, of course, the art of compromise in the face of many competing principles. "Eliminate gotchas" in this case directly opposes other principles like "no breaking changes", and "be consistent with other language features". Any thoughts you have on pros or cons of us taking this breaking change in a hypothetical future version of C\# would be greatly appreciated.

(This is part one of a two-part series on the loop-variable-closure problem. [Part two is here](http://blogs.msdn.com/b/ericlippert/archive/2009/11/16/closing-over-the-loop-variable-part-two.aspx).)

