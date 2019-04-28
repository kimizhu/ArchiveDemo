# Why are overloaded operators always static in C\#?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/14/2007 5:35:00 PM

-----

A language design question was posted to the Microsoft internal C\# discussion group this morning: "*Why must overloaded operators be static in C\#? In C++ an overloaded operator can be implemented by a static, instance or virtual method. Is there some reason for this constraint in C\#?*"

Before I get into the specifics, there is a larger point here worth delving into, or at least linking to. [Raymond Chen](http://blogs.msdn.com/oldnewthing/) immediately pointed out that the questioner had it backwards. The design of the C\# language is not a **subtractive** process; though I take [Bob Cringely's rather backhanded compliments](http://www.pbs.org/cringely/pulpit/2001/pulpit_20011101_000710.html) from 2001 in the best possible way, C\# is **not** Java/C++/whatever with the kludgy parts removed. Former C\# team member Eric Gunnerson wrote [a great article about how the process actually works](http://blogs.msdn.com/ericgu/archive/2004/01/12/57985.aspx).

Rather, the question we should be asking ourselves when faced with a potential language feature is "does the compelling benefit of the feature justify all the costs?" And costs are considerably more than just the mundane dollar costs of designing, developing, testing, documenting and maintaining a feature. There are more subtle costs, like, will this feature make it more difficult to change the type inferencing algorithm in the future? Does this lead us into a world where we will be unable to make changes without introducing backwards compatibility breaks? And so on.

In this specific case, the compelling benefit is small. If you want to have a virtual dispatched overloaded operator in C\# you can build one out of static parts very easily. For example:

 

public class B {  
    public static B operator+(B b1, B b2) { return b1.Add(b2); }  
    protected virtual B Add(B b2) { // ...

And there you have it. So, the benefits are small. But the costs are large. C++-style instance operators are weird. For example, they break symmetry. If you define an operator+ that takes a C and an int, then c+2 is legal but 2+c is not, and that badly breaks our intuition about how the addition operator should behave.

Similarly, with virtual operators in C++, the left-hand argument is the thing which parameterizes the virtual dispatch. So again, we get this weird asymmetry between the right and left sides. Really what you want for most binary operators is double dispatch -- you want the operator to be virtualized on the types of both arguments, not just the left-hand one. But neither C\# nor C++ supports double dispatch natively. (Many real-world problems would be solved if we had double dispatch; for one thing, [the visitor pattern becomes trivial](http://blogs.msdn.com/devdev/archive/2005/08/29/457798.aspx). My colleague [Wes](http://blogs.msdn.com/wesdyer/) is fond of pointing out that most design patterns are in fact necessary only insofar as the language has failed to provide a needed feature natively.)

And finally, in C++ you can only define an overloaded operator on a non-pointer type. This means that when you see c+2 and rewrite it as c.operator+(2), you are guaranteed that c is not a null pointer because it is not a pointer\! C\# also makes a distinction between values and references, but it would be very strange if instance operator overloads were only definable on non-nullable value types, and it would also be strange if c+2 could throw a null reference exception.

These and other difficulties along with the ease of building your own single (or double\!) virtual dispatch mechanisms out of static mechanisms, makes it easy to decide to not add instance or virtual operator overloading to C\#.

