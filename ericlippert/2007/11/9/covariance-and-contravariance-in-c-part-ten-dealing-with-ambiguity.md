<div id="page">

# Covariance and Contravariance in C\#, Part Ten: Dealing With Ambiguity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/9/2007 2:37:00 PM

-----

<div id="content">

<div class="mine">

OK, I wasn’t quite done. One more variance post\!

Smart people: suppose we made <span class="code">IEnumerable\<T\></span> covariant in <span class="code">T</span>. What should this program fragment do?

<span class="code"> </span>

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
3\) Always enumerate <span class="code">Giraffe</span>s.  
4\) Always enumerate <span class="code">Turtle</span>s.  
5\) Choose <span class="code">Giraffe</span>s vs <span class="code">Turtle</span>s at runtime.  
6\) Other, please specify.

And if you like any options other than (1), should this be a compile-time warning?

</div>

</div>

</div>

