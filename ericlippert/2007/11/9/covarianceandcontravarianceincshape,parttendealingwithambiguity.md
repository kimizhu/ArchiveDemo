# Covariance and Contravariance in C\#, Part Ten: Dealing With Ambiguity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/9/2007 2:37:00 PM

-----

OK, I wasn’t quite done. One more variance post\!

Smart people: suppose we made IEnumerable\<T\> covariant in T. What should this program fragment do?

 

class C : IEnumerable\<Giraffe\>, IEnumerable\<Turtle\> {  
    IEnumerator\<Giraffe\> IEnumerable\<Giraffe\>.GetEnumerator() {  
        yield return new Giraffe();  
    }  
    IEnumerator\<Turtle\> IEnumerable\<Turtle\>.GetEnumerator() {  
        yield return new Turtle();  
    }  
// \[etc.\]  
}  
   
class Program {  
    static void Main()  {  
        IEnumerable\<Animal\> animals = new C();  
        Console.WriteLine(animals.First().GetType().ToString());  
    }  
}

Options:

1\) Compile-time error.  
2\) Run-time error.  
3\) Always enumerate Giraffes.  
4\) Always enumerate Turtles.  
5\) Choose Giraffes vs Turtles at runtime.  
6\) Other, please specify.

And if you like any options other than (1), should this be a compile-time warning?

