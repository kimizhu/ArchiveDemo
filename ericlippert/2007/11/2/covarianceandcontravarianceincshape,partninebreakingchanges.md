# Covariance and Contravariance in C\#, Part Nine: Breaking Changes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/2/2007 10:33:00 AM

-----

Today in the last entry in my ongoing saga of covariance and contravariance I’ll discuss what breaking changes adding this feature might cause.

Simply adding variance awareness to the conversion rules should never cause any breaking change. However, the combination of adding variance to the conversion rules and making some types have variant parameters causes potential breaking changes.

People are generally smart enough to not write:

 

if (x is Animal)  
  DoSomething();  
else if (x is Giraffe)  
  DoSomethingElse(); // never runs

because the second condition is entirely subsumed by the first. But today in C\# 3.0 it is entirely sensible to write

 

if (x is IEnumerable\<Animal\>)  
  DoSomething();  
else if (x is IEnumerable\<Giraffe\>)  
  DoSomethingElse();

because there did not used to be any conversion between IEnumerable\<Animal\> and IEnumerable\<Giraffe\>. If we turn on covariance in IEnumerable\<T\> and the compiled program containing the fragment uses the new library then its behaviour when given an IEnumerable\<Giraffe\> will change. The object will be assignable to IEnumerable\<Animal\>, and therefore the “is” will report “true”.

There is also the issue of existing source code changing semantics or turning compiling programs into erroneous programs. For example, overload resolution may now fail where it used to succeed. If we have:

 

interface IBar\<T\>{} // From some other assembly  
...  
void M(IBar\<Tiger\> x){}  
void M(IBar\<Giraffe\> x){}  
void M(object x) {}  
...  
IBar\<Animal\> y = whatever;  
M(y);

Then overload resolution picks the object version today because it is the sole applicable choice. If we change the definition of IBar to

 

interface IBar\<-T\>{}

and recompile then we get an ambiguity error because now all three are applicable and there is no unique best choice.

We always want to avoid breaking changes if possible, but sometimes new features are sufficiently compelling and the breaks are sufficiently rare that it’s worth it. My intuition is that by turning on interface and delegate variance we would enable many more interesting scenarios than we would break.

What are your thoughts? Keep in mind that we expect that the vast majority of developers will never have to define the variance of a given type argument, but they may take advantage of variance frequently. Is it worth our while to invest time and energy in this sort of thing for a hypothetical future version of the language?

