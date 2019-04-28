# An Inheritance Puzzle, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/27/2007 9:36:00 AM

-----

Once more I have returned from my ancestral homeland, after some weeks of sun, rain, storms, wind, calm, friends and family. I could certainly use another few weeks, but it is good to be back too.

Well, enough chit-chat; back to programming language design. Here's an interesting combination of subclassing with nesting. Before trying it, what do you think this program *should* output?

 

public class A\<T\> {  
    public class B : A\<**int**\> {  
        public void M() {  
            System.Console.WriteLine(typeof(T).ToString());  
        }  
        public class C : B { }  
    }  
}  
class MainClass {  
    static void Main() {  
        A\<**string**\>.B.C c = new A\<**string**\>.B.C();  
        c.M();  
    }  
}

Should this say that T is int, string or something else? Or should this program not compile in the first place?

It turned out that the actual result is not what *I* was expecting at least. I learn something new about this language every day.

Can you predict the behaviour of the code? Can you justify it according to the specification? (The specification is really quite difficult to understand on this point, but in fact it does all make sense.)

(The answer to the puzzle is [here](http://blogs.msdn.com/b/ericlippert/archive/2007/07/30/an-inheritance-puzzle-part-two.aspx).)

