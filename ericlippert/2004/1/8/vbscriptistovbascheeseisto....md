# VBScript is to VB as Cheese is to...?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/8/2004 12:48:00 PM

-----

A [Joel On Software](http://www.joelonsoftware.com/) reader made the mistake of stating in my presence: 

"VB is to VBScript as Java is to JavaScript." 

Though I can see some truth in that, I'd classify that as a poor analogy. 

Why?  Because VBScript was designed to be a simple subset of the VB language. One of our major design criteria was that a VBScript program be a legal VB program. There are some exceptions, but more or less that's the case. 

JavaScript is not a subset of Java at all. It's an entirely different language with a superficially similar syntax -- just as C, C++, C\# (and for that matter, perl and AWK\!) have superficially similar syntaxes.  Just because you can write z = x \<\< y + a & b; in a bunch of different languages doesn't mean they have anything else in common. Remember, JavaScript was originally called LiveScript, and Netscape changed the name because (among other reasons) they believed that they could capitalize on the incredible hype surrounding Java. Unfortunately, they managed to create a lot of confusion as well. 

Compare JavaScript and Java at a high level, not a syntax level, and you'll see that the type system, exception system, class system, libraries, compilation model are all quite different. Pretty much everything about Java and JavaScript that makes them interesting as languages languages are different\!  JavaScript is certainly more similar to Java than, say, Prolog, or Befunge, or Urdu, or the colour blue, but that analogy really doesn't hold up well. 

Finally, as my friend Jon once remarked "American cheese is to good cheese as this analogy is to, uh, something."  I puzzled over that one for some time trying to determine if it is an apt analogy or not.  Like American cheese, it's a paradox.

