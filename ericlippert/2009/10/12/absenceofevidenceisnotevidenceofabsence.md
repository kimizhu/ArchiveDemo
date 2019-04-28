# Absence of evidence is not evidence of absence

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/12/2009 6:51:00 AM

-----

Today, two more subtly incorrect myths about C\#.

As you probably know, C\# requires all local variables to be explicitly assigned before they are read, but assumes that all class instance field variables are initially assigned to default values. An explanation of why that is that I sometimes hear is "*the compiler can easily prove that a local variable is not assigned, but it is much harder to prove that an instance field is not assigned. And since the class's default constructor automatically assigns all instance fields to default values, you don't need to do the analysis for fields."*

Both statements are subtly incorrect.

The first statement is incorrect because the compiler in fact cannot and does not prove that a local variable is not assigned. Proving that is (1) impossible, and (2) does not give us any useful information we can act upon. It's impossible because proving that a given variable is assigned a value is equivalent to solving the Halting Problem:

 

int x;  
if (/\*condition requiring solution of the halting problem here\*/) x = 10;  
print(x);

If what we wanted to do was prove that x was *unassigned* then we would have to *at compile time* prove that the condition was false. Our compiler is not that sophisticated\!

But the deeper point here is that we're not interested in proving for certain that x is *unassigned*. We're interested in proving for certain that x is *assigned*\! If we can prove that for certain, then x is "definitely assigned". If we cannot prove that for certain then x is "not definitely assigned". We're only interested in "definitely unassigned" insofar as "definitely unassigned" is a stronger version of "not definitely assigned". If x is read from when it is "not definitely assigned", that's a bug.

That is, we're attempting to prove that x is assigned, and our failure to prove that at every point where it is read is what motivates the error. That failure could be because of a bona fide bug in your program, or it could be because our flow analyzer is extremely conservative. For example:

 

int x, y = 0;  
if (0 \* y == 0) x = 10;  
print(x);

You and I know that x is definitely assigned, but in C\# 3 the compiler is *deliberately* not smart enough to prove that. (Interestingly enough, it *was* smart enough in C\# 2. [I broke that to bring the compiler into line with the spec; being smarter but in violation of the spec is not necessarily a good thing](http://blogs.msdn.com/ericlippert/archive/2006/03/29/the-root-of-all-evil-part-two.aspx).) 

This example again shows that we do not prove that x is unassigned; if we did prove that, then clearly our prover would contain an error, since you and I both know that x is definitely assigned. Rather, we fail to prove that x is assigned.

This is an interesting twist on the believers vs skeptics argument that goes like this: the skeptic says "there's no reliable evidence that bigfoot exists, therefore, bigfoot does not exist". The believer says "absence of reliable evidence is not itself evidence of absence; and yes, bigfoot does exist". In both cases, reasoning from a position of lacking reliable evidence is seldom good reasoning\! But in our case, it is precisely *because* we lack reliable evidence that we are coming to the conclusion that we do not know enough to allow you to read from x.

(The relevant principle for tentatively concluding that bigfoot is mythical based on a lack of reliable evidence is "extraordinary claims require extraordinary evidence". *It* *is reasonable to assume that an extraordinary claim is false until reliable evidence is produced*. When overwhelmingly reliable evidence is produced of an extraordinary claim -- say, the extraordinary claim that time itself slows down when you move faster -- then it makes sense to believe the extraordinary claim. Overwhelming evidence has been provided for the theory of relativity, but not for the theory of bigfoot.)

The second myth is that the default constructor of a class initializes the fields to their default values. This can be shown to be false by several arguments.

First, a class need not have a default constructor, and yet its fields are always observed to be initially assigned. If there is no default constructor, then something else must be initializing the fields.

Second, even if a class does have a default constructor, there's no guarantee that it will be called. Some other constructor could be called.

Third, the field initializers of a class run before any constructor body runs, therefore it cannot be the constructor body that does the initialization; that would be wiping out the results of the field initializers.

Fourth, constructors can call other constructors; if each of those constructors was initializing the fields to zero, then that would be wasteful; we'd be unneccessarily re-initializing already-wiped-out fields.

What actually happens is that the CLI memory allocator guarantees that the memory allocated for a given class instance will be initialized to all zeros before the constructor is called. By the time the constructors run the object is already freshly zeroed out and ready to go.

