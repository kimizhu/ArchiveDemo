# No Backtracking, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/7/2010 10:12:00 AM

-----

As i was saying last time, the nice thing about "no backtracking" is that it makes the language much easier to understand. Simple rules benefit both the compiler and the code reader; both are attempting to read the code to make sense of it. It is not always a good idea to take advantage of the compiler's ability to search a large space if that makes it harder for a human to understand the code.

Suppose you have something like this mess: (\*) namespace XYZ.DEF  
{   
    public class GHI {}   
}   
namespace QRS.DEF.GHI   
{   
    public class JKL { }   
}  

... in another file ... 

using QRS;   
namespace TUV    
{  
    using XYZ;  
    namespace ABC  
    {  
        namespace DEF  
        {  
            class GHI { }  
            class MNO : DEF.GHI.JKL { }  
        }  
    }  
}  

And now we must work out the base type of MNO. With no backtracking we say "The nearest container of MNO that has a member DEF is ABC, therefore DEF means ABC.DEF". Therefore GHI is ABC.DEF.GHI. Therefore JKL is ABC.DEF.GHI.JKL, which does not exist, therefore, give an error. The developer must fix the error by giving a type name that lets the compiler identify which DEF you meant. If we had backtracking, what would we have to do? We’d get that error, and then we’d backtrack. Does XYZ contain a DEF? Yes. Does it contain a GHI? Yes. Does it contain a JKL? No. **Backtrack again.** Does QRS contain an DEF.GHI.JKL? Yes. That *works*, but can we logically conclude from the fact that it works that it is the one the user *meant*? Who the heck knows in this crazy situation? We got all kinds of good bindings in there that then went bad very late in the game. The idea that we stumbled upon the *desired* answer after going down so many blind alleys seems highly suspect. Maybe there is yet another choice in there that is the one the user meant. We cannot know that unless we try *all* of them, and again, that could involve a lot of searching. The correct thing to do here is not to backtrack multiple times and try out all kinds of worse bindings for every stage of the lookup. The correct thing to do is to say "buddy, the best possible match for this lookup gives nonsensical results; give me something less ambiguous to work with here please." An unfortunate fact about writing a language where the compiler *by design* complains loudly if the best match is something that doesn't work, is that developers frequently say "well, sure, in *general* I want the compiler to point out all my mistakes -- or, rather, *all my coworker's mistakes*. But for this *specific* case, I know what I am doing, so please, compiler, **do what I mean, not what I say**." Trouble is, you can't have it both ways. You can't have both a compiler that both enforces rigid rules that make it highly likely that suspicious code will be aggressively identified as erroneous *and* allow crazy code via compiler heuristics that figure out "what I really meant" when you write something that the compiler quite rightly sees as ambiguous or wrong. I could discuss many more places where we could do backtracking but do not. Method type inference, for example, always either makes progress or fails; it never backtracks in C\# (\*\*). But I think I will leave it at that. Except to say that there is one place where we do use backtracking, and that’s analysis of overload resolution in nested lambdas. I wrote about that [here](http://blogs.msdn.com/b/ericlippert/archive/2007/03/26/lambda-expressions-vs-anonymous-methods-part-four.aspx). (\*) When reading this remember that in C\#, “namespace XYZ.DEF { }” is a short form for “namespace XYZ{ namespace DEF { } }”

(\*\*) Unlike in, say, F\#.

