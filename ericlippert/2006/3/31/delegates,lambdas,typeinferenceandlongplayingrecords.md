# Delegates, Lambdas, Type Inference and Long Playing Records

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/31/2006 1:17:00 PM

-----

Today is my 33 1/3rd birthday\! I'm one third of the way through my first century. I feel like I should go buy some LPs to celebrate, not that I have anything that will play them.

In other news, someone sent me this code the other day. Suppose you've got a collection of mammals and you want to feed all the giraffes:

delegate void Action\<X\>(X x);  
class MyCollection\<T\> : Collection\<T\> where T : class {  
  public void Apply\<S\>(Action\<S\> action) where S : class, T {  
    foreach ( T t in this ) {  
      S s = t as S;  
      if (s \!= null )  
        action(s);  
    }  
  }  
}  
//...  
  
MyCollection\<Mammal\> mammals = new MyCollection\<Mammal\>();  
//...  
mammals.Apply\<Giraffe\>(delegate(Giraffe g){Feed(g);});  

This works just fine. The questioner was wondering why this didn't work:

mammals.Apply(delegate(Giraffe g){Feed(g);});  

That is, why do we need to specify the type parameter to the generic method? Shouldn't type inference take care of it?

That would be nice, but unfortunately section 26.6.4 of the specification clearly states that nothing is inferred from any anonymous method, null, or method group argument.

In the next version of C\# we will have a new, more powerful, more flexible kind of anonymous method called a lambda expression. A lambda expression is like an anonymous method with additional type inferencing features and a more natural syntax than the clunky delegate syntax.

Unfortunately for the questioner, the new lambda type inferencing rules actually will not help with this scenario. However, the new type inferencing rules will apply in any scenario where "classic" type inferencing can be used to determine the generic type of a lambda argument, and from that, determine the generic type of a lambda return, and from that, the generic return type of a delegate, and from that, a method type parameter.

That last sentence probably made no sense. Let's look at an example where the new type inferencing rule would apply.

Suppose you have a collection of Customers and you want to create a collection of their names:

delegate R Func\<A, R\> (A arg);  
IEnumerable\<S\> Select\<T, S\>(IEnumerable\<T\> source, Func\<T, S\> selector) {...}  
//...  
names = Select(customers, c =\> c.Name);  

With the old type inferencing rules we can infer that T is Customer, and we would infer nothing from the lambda. However, in C\# 3.0 we will enter a second round of type inferencing, where we'd note that we hadn't inferred anything from the lambda yet, and that it is being converted to a partially inferred delegate type Func\<Customer, R\> requiring type inference on the return type.

We now can infer that the parameter to the lambda, c must be of type Customer, and therefore the return type of the lambda must be the type of Customer.Name, which is string. Therefore the delegate type is Func\<Customer, string\>, and hey,we've managed to infer the types of all the parameters to Select\<T, S\>, so we're done.

Cool eh? Now all we've got to do is actually implement it...

