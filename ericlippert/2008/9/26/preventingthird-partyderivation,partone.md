# Preventing third-party derivation, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/26/2008 11:17:00 AM

-----

In this episode of FAIC, a conversation I had with a C\# user recently. Next time, some further thoughts on how to use the CLR security system in this sort of scenario.

***Him: I have this abstract class defined in one assembly:***

 

// Assembly FooBar.DLL  
public abstract class Foo  
{  
    internal abstract void DoSomething();  
// ...  
}

***I want to create a concrete class derived directly from Foo in another assembly, Blah.DLL.***

Me: You are going to have to learn to live with disappointment.

***Him: But I should be able to because I have public access to the class Foo\!***

Me: That statement is false. Having public access to a class absolutely does not mean that it is always legal to create a subclass in another assembly. There are any number of reasons why it might not be legal to create a subclass in another assembly; you happen to have run across just one of them. (There are plenty more -- for example, you could make a public class and mark all the constructors as internal. It's not possible to subclass that from another assembly either, since the derived class constructor does not have an accessible base class constructor that it can call.)

***Him: You're right. In this case, I can’t provide an overriding implementation for the internal abstract method DoSomething() because it is not accessible. Therefore the compiler will give an error when I try to subclass Foo. How do I get around this?***

Me: You don’t get around it. The type safety system is working as designed; don’t try to defeat it.

If you own the FooBar.DLL assembly then of course there are several ways you can do what you want. Two that immediately come to mind are (1) mark Blah.DLL as a friend assembly of FooBar.DLL using the InternalsVisibleTo attribute (2) change the accessibility of DoSomething to public, protected or protected internal.

***Him: Why is it even allowed to have a nonextensible class like this in the first place?***

Me: It is sometimes a good thing to make a class that cannot be extended arbitrarily. I have myself occasionally used a similar technique to ensure that no third party can subclass one of my public base classes, though, as we’ll see later, there are other, perhaps better ways.

Look at it this way: if the author of the class *wanted* you to be able to subclass it, they probably would have made it *possible.* Clearly they do not want you to subclass this class.

***Him: My previous question has been thoroughly begged. Why would someone want to prevent a third party from subclassing?***

Me: I can think of a few reasons. Here are two:

1\) Designing a class to be extended effectively by third parties is work, and work requires effort. If the class is not intended to be extended by third parties, but must be unsealed (for internal extension, for example) then the implementer is faced with a choice: spend the time/money/effort unnecessarily, provide an extensible but brittle class, or prevent extension by some other method.

This trick is a cheap, easy and straightforward way to prevent extension by arbitrary third parties *without preventing extension by internal classes*. As we'll see next time, there are other ways you can do this too.

2\) A class may need to be extensible internally and used externally, but must not be extended by third parties for security reasons. Imagine you have a method Transfer which takes two instances of your class BankAccount. Suppose BankAccount is an abstract class with two derived classes, CaymanIslandsAccount and SwissAccount. Do you really want arbitrary third parties able to make new objects which fulfill the BankAccount contract but were not implemented by you?

Again, you end up with a tradeoff – either implement Transfer so that it does type checks on the BankAccount passed to it (possibly expensive both in initial implementation and maintenance), implement Transfer so that it accepts any old thing (dangerous\!), or prevent anyone from making an unknown kind of BankAccount (cheap, safe).

In general, good security design is “make them do something impossible in order to break the system”. (Or even better “make them do seven impossible things”.) By making it impossible for third parties to fulfill the contract, you add additional security to the system. (By no means *enough* security, but a little bit *more*.)

***Him: Thanks, I see what's going on here. Clearly I got into this situation because this trick of using access modifiers to prevent extension is insufficient to convey the author of FooBar.DLL's intentions.***

Me: Next time we'll talk about more obvious ways to state the intention of "please don't subclass this thing".

