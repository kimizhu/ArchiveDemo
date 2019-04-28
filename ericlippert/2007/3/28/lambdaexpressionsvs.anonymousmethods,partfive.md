# Lambda Expressions vs. Anonymous Methods, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/28/2007 10:45:00 AM

-----

[Last time](http://blogs.msdn.com/ericlippert/archive/2007/03/26/lambda-expressions-vs-anonymous-methods-part-four.aspx) I demonstrated that the compiler could have to do an exponential number of bindings in order to determine whether there was a unique best overload resolution for a function call that takes a lambda. Some of you may have wondered whether we simply were not being clever enough in the compiler. Perhaps there is some way to optimize this problem so that the unique solution is found in less than exponential time.

As it turns out, the question of whether there is a clever way to do this in C\# 3.0 is equivalent to solving the most famous unsolved problem in computer science. That is a problem which has stumped generations of the finest minds in academia, and is widely believed to be unsolvable, so I don't feel *particularly* bad about not solving this one myself.

Consider the following *slight* revision of the set of overloads I sketched out last time:

 

class MainClass  
{  
    class T{}  
    class F{}  
    delegate void DT(T t);  
    delegate void DF(F f);  
    static void M(DT dt)  
    {  
        System.Console.WriteLine("true");  
        dt(new T());  
    }  
    static void M(DF df)  
    {  
        System.Console.WriteLine("false");  
        df(new F());  
    }  
    static T Or(T a1, T a2, T a3){return new T();}  
    static T Or(T a1, T a2, F a3){return new T();}  
    static T Or(T a1, F a2, T a3){return new T();}  
    static T Or(T a1, F a2, F a3){return new T();}  
    static T Or(F a1, T a2, T a3){return new T();}  
    static T Or(F a1, T a2, F a3){return new T();}  
    static T Or(F a1, F a2, T a3){return new T();}  
    static F Or(F a1, F a2, F a3){return new F();}  
    static T And(T a1, T a2){return new T();}  
    static F And(T a1, F a2){return new F();}  
    static F And(F a1, T a2){return new F();}  
    static F And(F a1, F a2){return new F();}  
    static F Not(T a){return new F();}  
    static T Not(F a){return new T();}  
    static void MustBeT(T t){}  
    static void Main()  
    {  
        // Introduce enough variables and then encode any Boolean predicate:  
        // eg, here we encode (\!x3) & ((\!x1) & ((x1 | x2 | x1) & (x2 | x3 | x2)))  
        M(x1=\>M(x2=\>M(x3=\>MustBeT(  
          And(  
            Not(x3),   
            And(  
              Not(x1),   
              And(  
                Or(x1, x2, x1),   
                Or(x2, x3, x2))))))));  
    }  
}

This expression *makes the compiler solve the [Boolean satisfiability problem](http://en.wikipedia.org/wiki/Boolean_satisfiability_problem)* with three terms per "or", aka "3SAT". If 3SAT has a unique solution then the program compiles and produces the unique solution. If it has more than one solution then compilation fails with an ambiguity error. If it has no solution then the program fails with the error that MustBeT has an F argument. But no matter how you slice it, the compiler must solve SAT.

Determining whether a given predicate has a set of valuations for its variables which make it true is an [NP-complete](http://en.wikipedia.org/wiki/NP-complete) problem; actually *finding* the set of valuations is an [NP-hard](http://en.wikipedia.org/wiki/NP-hard) problem. Therefore, *overload resolution in C\# 3.0 is at the very least NP-hard*. There is no known polynomial-time algorithm for any NP-complete or NP-hard problem and it is widely believed that there is none to be found, though that conjecture has yet to be proven. So our dream of fast lambda type analysis in C\# 3.0 is almost certainly doomed, at least as long as we have the current rules for overload resolution.

Does this really matter in practice? As I mentioned last time, hopefully not. Other languages also have this issue. When I ran an early draft of this post past him, [Erik Meijer](http://research.microsoft.com/~emeijer/) immeditely pointed out that the [ML](http://en.wikipedia.org/wiki/ML_programming_language) type inference system has [similar worst cases](http://portal.acm.org/citation.cfm?id=96748&coll=portal&dl=ACM), and the [Haskell](http://en.wikipedia.org/wiki/Haskell_%28programming_language%29) type inference system is even worse. Apparently in Haskell you can encode a Turing machine into the type system and make the compiler run it\! In practice these sorts of problems do not arise in real-world code in any of these languages, so I am not too worried about it for C\# 3.0.

