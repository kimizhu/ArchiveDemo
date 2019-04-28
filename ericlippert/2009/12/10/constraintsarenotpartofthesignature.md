# Constraints are not part of the signature

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/10/2009 9:31:00 AM

-----

What happens here?  

class Animal { }  
class Mammal : Animal { }  
class Giraffe : Mammal { }  
class Reptile : Animal { }  
…  
static void Foo\<T\>(T t) where T : Reptile { }  
static void Foo(Animal animal) { }  
static void Main()  
{  
    Foo(new Giraffe());  
}

Most people assume that overload resolution will choose the second overload. In fact, this program produces a compile error saying that T cannot be Giraffe. Is this a compiler bug?

No, this behaviour is correct according to the spec. First we attempt to determine the candidate set. Clearly the second overload is a member of the candidate set. The first overload is a member of the candidate set if type inference succeeds.

The method type inference algorithm considers **only** whether the method type arguments can be consistently inferred from the types of the arguments. Method type inference cares not a bit about whether the resulting method is malformed in some other way. Its only job is to work out the best possible type arguments given the arguments to the method. Clearly the best type for T is Giraffe, so that’s what we infer.

So we now have two methods in the candidate set, Foo\<Giraffe\>, and the second overload. Which is better?

Again, overload resolution looks only at the arguments you passed in, and compares them to the types of all the candidates. The argument is of type Giraffe. We have a choice: argument of type Giraffe goes to parameter of type Giraffe, or argument of type Giraffe goes to parameter of type Animal. Clearly the former is better; it’s an exact match.

Therefore we discard the second overload because it is worse than another candidate. That leaves one candidate left, which is an exact match. Only then, after overload resolution, do we check to see whether the generic constraints are violated.

When I try to explain this to people, they often bring up this portion of the specification as evidence that I am wrong, wrong, wrong:

 

> If F is generic and M has no type argument list, F is a candidate when:  
> 1\) type inference succeeds, and  
> 2\) once the inferred type arguments are substituted for the corresponding method type parameters, *all constructed types in the parameter list of F satisfy their constraints*, and the parameter list of F is applicable with respect to A. \[emphasis added\]

This appears at first glance to say that Foo\<Giraffe\> cannot be a candidate because the constraints are not satisfied. That is a misreading of the specification; the bit “in the parameter list” is referring to the *formal* *parameter* list, not the *type parameter* list.

Let me give you an example of where this rule comes into play, so that it’s clear. Suppose we have

 

class C\<T\> where T : Mammal {}  
…  
static void Bar\<T\>(T t, C\<T\> c) where T : Mammal {}  
static void Bar(Animal animal, string s) { }  
…  
Bar(new Iguana(), null);

Type inference infers that Bar\<Iguana\> might be a candidate. But that would mean that we are calling a method that converts null to C\<Iguana\>, which violates the constraint in the declaration of C\<T\>. Therefore the results of type inference are discarded, and Bar\<Iguana\> is not added to the candidate set.

So if that’s not the relevant bit of the spec, what is? The relevant bit happens *after* the best method has been determined, not before.

 

> If the best method is a generic method, the type arguments (supplied or inferred) are checked against the constraints declared on the generic method. If any type argument does not satisfy the corresponding constraints on the type parameter, a compile-time error occurs.

It is often surprising to people that an invalid method can be chosen as the best method, chosen over a less good method that would be valid. (\*) **The principle here is overload resolution (and method type inference) find the best possible match between a list of arguments and each candidate method’s list of formal parameters.** That is, they look at the *signature* of the candidate method. If the best possible match between the arguments and the signature of the method identify a method that is for whatever reason not possible to call, then *you need to choose your arguments more carefully so that the bad thing is no longer the best match*. We figure that you want to be told that there's a problem, rather than silently falling back to a less-good choice.

UPDATE: My Twilight-reading friend Jen from a couple episodes back points out that this is not just good language design, it's good dating advice. **If the best possible match between your criteria and an available guy on Match.com identifies a guy who, for whatever reason, is impossible to call, then you need to choose your criteria more carefully so that the bad guy is no longer the best match.**  Words to live by.

-----

(\*) I already discussed the related situation of how it is the case that [we can choose a static method over an instance method even when there’s no hope of the static method being the right one.](http://blogs.msdn.com/ericlippert/archive/2009/07/06/color-color.aspx)

