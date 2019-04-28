# Odious ambiguous overloads, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2006 11:58:00 AM

-----

As you might have gathered, a lot of the decisions we have to make day-to-day here involve potential breaking changes on odd edge cases. Here's another one.

Consider the following terrible but legal code:

 

public interface I1\<U\> {  
    void M(U i);  
    void M(int i);  
}

My intense pain begins when the user writes:

 

public class C1 : I1\<int\> {

In the early version of the C\# specification it was actually illegal to have an ambiguity like this, but the spec was changed so that when doing overload resolution on a call to M we can choose one, according to section 14.4.2.2:

*If one \[argument type\] is non-generic, but the other is generic, then the non-generic is better.*

But still, as you will soon see, we're in a world of hurt for another reason, namely that this class must now implement both M(int i) and, uh, M(int i). Fortunately, a nonexplicit implementation binds to both methods, so this works just fine:

 

public class C1 : I1\<int\> {  
    public void M(int i) {  
        Console.WriteLine("class " + i);  
    }  
}

The method implements both versions of M and the contract is satisfied.  But we have problems if we try to do an explicit interface implementation:

 

public class C2 : I1\<int\> {  
    void I1\<int\>.M(int i) {  
        Console.WriteLine("explicit " + i);  
    }  
}

Does this explicitly implement both members of I1?  Or just one?  If so, which one?

In the current compiler this code produces a terrible, terrible error:

 

error CS0535: 'C2' does not implement interface member 'I1\<int\>.M(int)'

Is that so?  It sure looks like it implements it\!

What happens when we have both an explicit implementation and a class implementation?  **The spec does not actually say what to do.** It turns out that we end up in a situation where runtime behaviour depends on **source code order** of the interface\! Check this out:

 

public interface I1\<U\> {  
    void M(U i); // generic first  
    void M(int i);  
}

public interface I2\<U\> {  
    void M(int i);  
    void M(U i); // generic second  
}

public class C3: I1\<int\>, I2\<int\> {  
    void I1\<int\>.M(int i) {  
        Console.WriteLine("c3 explicit I1 " + i);  
    }  
    void I2\<int\>.M(int i) {  
        Console.WriteLine("c3 explicit I2 " + i);  
    }  
    public void M(int i) {  
        Console.WriteLine("c3 class " + i);  
    }  
}

class Test {  
    static void Main() {  
        C3 c3 = new C3();  
        I1\<int\> i1\_c3 = c3;  
        I2\<int\> i2\_c3 = c3;  
        i1\_c3.M(101);  
        i2\_c3.M(102);  
    }  
}

What happens here is that the explicit interface implementation mappings in the class match the methods in the interfaces in a first-come-first-served manner:

void I1\<int\>.M(U) maps to explicit implementation void I1\<int\>.M(int i)  
void I1\<int\>.M(int) maps to implicit implementation public void M(int i)  
void I2\<int\>.M(int) maps to explicit implementation void I2\<int\>.M(int i)  
void I2\<int\>.M(U) maps to implicit implementation public void M(int i)

Then (because of the aforementioned section 14.4.2.2) when we see

 

        i1\_c3.M(101);  
        i2\_c3.M(102);

we prefer the typed-as-int versions to the generic substitution versions, so this program calls the two non-generic versions and produces the output:

 

c3 class 101  
c3 explicit I2 102

And as you'd expect, if we force the compiler to pick the generic versions then we get similar behaviour:

 

static void Main() {  
    C3 c3 = new C3();  
    Thunk1\<int\>(c3,103);  
    Thunk2\<int\>(c3, 104);  
}  
static void Thunk1\<U\>(I1\<U\> i1, U u) {  
    i1.M(u);  
}  
static void Thunk2\<U\>(I2\<U\> i2, U u) {  
    i2.M(u);  
}

The binding of the overload resolution in the thunk bodies happens before the substitution of the type parameters, so these always bind to the generic versions of the methods. As you would expect from the mappings above, this outputs

 

c3 explicit I1 103  
c3 class 104

Again, this shows that source code order has an unfortunate semantic import.

Given this unfortunate situation -- no spec guidance and an existing implementation that behaves strangely -- what would you do? (Of course "do nothing" is an option.) I'm interested to hear your ideas, and I'll describe what we actually did next time.

