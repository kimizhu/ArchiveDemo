# Why Can't I Access A Protected Member From A Derived Class? Part Six

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/14/2010 6:42:00 AM

-----

Reader Jesse McGrew asks an excellent follow-up question to [my 2005 post about why you cannot access a protected member from a derived class](http://blogs.msdn.com/ericlippert/archive/tags/protected/default.aspx). (You probably want to re-read that post in order to make sense of this one.)

I want to be clear in my terminology, so I’m going to define some terms. Suppose we have a call foo.Bar() inside class C. The value of foo is the “receiver” of the call. The compile-time type of foo is the “compile time type of the receiver”. The “runtime type of the receiver” could potentially be more derived. The “call site type” is C.

The rule in C\# is that if Bar is protected, then the compile-time type of the receiver must be the call site type, or a type derived from the call site type.  It cannot be a base type of the call site type, because then the runtime type of the receiver might have a “cousin” (or sibling, or uncle… but let’s not split hairs, let’s just call them all cousins) relationship to the call site type, rather than an “ancestor/descendant” relationship.

Jesse quite rightly points out that my original answer to the question was not really complete. There are two questions unanswered:

1\) Why would it be a bad thing to allow calling a protected method from a receiver whose runtime type is a “cousin” class?

2\) Supposing for the sake of argument that there is a good answer to (1) – why doesn’t that argument apply equally well to calling a protected method via a receiver whose compile-time type is the same as the call site type? The runtime type of the receiver still could be more derived.

I’ll answer the first question first. In this answer I’m going to use a humorous and exaggerated example to illustrate the problem; I want to emphasize that this is **not** how you should actually design applications that need “big S” Security, like bank accounts. What I want to illustrate here is simply that allowing protected methods to be called via “cousins” makes it difficult to maintain invariants and therefore difficult to write correct, robust code.

 

// Good.dll:  
  
    public abstract class BankAccount  
    {  
      abstract protected void DoTransfer(  
        BankAccount destinationAccount,   
        User authorizedUser,  
        decimal amount);  
    }  
    public abstract class SecureBankAccount : BankAccount  
    {  
      protected readonly int accountNumber;  
      public SecureBankAccount(int accountNumber)  
      {  
        this.accountNumber = accountNumber;  
      }  
      public void Transfer(  
        BankAccount destinationAccount,   
        User authorizedUser,  
        decimal amount)  
      {  
        if (\!Authorized(user, accountNumber)) throw something;  
        this.DoTransfer(destinationAccount, user, amount);  
      }  
    }  
    public sealed class SwissBankAccount : SecureBankAccount  
    {  
      public SwissBankAccount(int accountNumber) : base(accountNumber) {}  
      override protected void DoTransfer(  
        BankAccount destinationAccount,   
        User authorizedUser,  
        decimal amount)  
      {  
        // Code to transfer money from a Swiss bank account here.  
        // This code can assume that authorizedUser is authorized.  
        // We are guaranteed this because SwissBankAccount is sealed, and  
        // all callers must go through public version of Transfer from base  
        // class SecureBankAccount.  
      }  
    }  
    // Evil.exe:  
    class HostileBankAccount : BankAccount  
    {  
      override protected void Transfer(  
        BankAccount destinationAccount,   
        User authorizedUser,  
        decimal amount) {  }  
      public static void Main()  
      {  
        User drEvil = new User("Dr. Evil");  
        BankAccount yours = new SwissBankAccount(1234567);  
        BankAccount mine = new SwissBankAccount(66666666);  
        yours.DoTransfer(mine, drEvil, 1000000.00m); // compilation error  
        // You don't have the right to access the protected member of  
        // SwissBankAccount just because you are in a  
        // type derived from BankAccount.  
      }  
    }

Dr. Evil's attempt to steal [ONE... MILLION... DOLLARS](https://www.youtube.com/watch?v=cKKHSAE1gIs)... from your Swiss bank account has been foiled by the C\# compiler. Obviously this is a silly example, and obviously, fully-trusted code could do anything it wants to your types -- fully-trusted code can start up a debugger and change the code as its running. Full trust means *full trust*. Again, do not actually design a real security system this way\!  But my point is simply that the "attack" that is foiled here is **someone attempting to do an end-run around the invariants set up by SecureBankAccount in order to access the code in SwissBankAccount directly.** The second question is "Why doesn't SecureBankAccount also have this restriction?"  In my example, SecureBankAccount says “this.DoTransfer(destinationAccount, user, amount);” Clearly "this" is of type SecureBankAccount or something more derived. It could be any value of a more derived type, including a new SwissBankAccount. Doesn’t the same concern apply? **Couldn't SecureBankAccount be doing an end-run around SwissBankAccount's invariants?** Yes, absolutely\! And because of that, *the authors of SwissBankAccount are required to understand and approve of everything that their base class does\!*  You can't just go deriving from some class willy-nilly and hope for the best. The implementation of your base class is allowed to call the set of protected methods exposed by the base class. If you want to derive from it then you are required to read the documentation for that class, or the code, and understand under what circumstances your protected methods will be called, and write your code accordingly. **Derivation is a way of sharing implementation details; if you don't understand the implementation details of the thing you are deriving from then don't derive from it.** And besides, the base class is always written *before* the derived class. The base class isn't up and changing on you, and presumably you trust the author of the class to not attempt to break you sneakily with a future version. (Of course, a change to a base class can always cause problems; this is yet another version of the [brittle base class](http://blogs.msdn.com/ericlippert/archive/tags/Brittle+Base+Classes/default.aspx) problem.) The difference between the two cases is that when you derive from a base class, you have the behaviour of *one* class *of your choice* to understand and approve of. That is a tractable amount of work. The authors of SwissBankAccount are required to precisely understand what SecureBankAccount guarantees to be invariant before the protected method is called. But they should not have to understand and trust every possible behaviour of every possible cousin class that just happens to be derived from the same base class. Those guys could be implemented by anyone and do anything. You would have no ability whatsoever to understand any of their pre-call invariants, and therefore you would have no ability to successfully write a working protected method. Therefore, we save you that bother and disallow that scenario. And besides, we *have* to allow you to call protected methods on receivers of potentially more-derived classes. Suppose we didn't allow that and deduce something absurd. Under what circumstances could a protected method ever be called, if we disallowed calling protected methods on receivers of potentially-more-derived classes?  The only time you could ever call a protected method in that world is if you were calling your own protected method from a sealed class\! Effectively, protected methods could almost never be called, and the implementation that was called would always be the most derived one. What's the point of "protected" in that case? Your "protected" means the same thing as "private, and can only be called from a sealed class". That would make them rather less useful. So, the short answer to both questions is "because if we didn't do that, it would be impossible to use protected methods at all."  We restrict calls through less-derived receiver compile-time types because if we don't, it's impossible to safely implement any protected method that depends on an invariant.  We allow calls through potential subtypes because if we do not allow this, then we don't allow hardly any calls at all. Finally, an unasked question: what if you are the person writing the base class with a protected method? Essentially you are in the same boat as the person writing any *public* or *virtual* method; then you have to accept that anyone can call your public method in any way it chooses, or that derived class can override your virtual method and make it do something completely different, at their discretion. **If you write a protected method, you have to accept that any derived class can call that method in any way it chooses and write the base class code accordingly.** When you write a public method, you have to consider the consequences of bad callers; if there are ways that bad callers can misuse your public method, then you need to consider the cost to the user vs the cost of hardening the method against the potentially bad caller. The same is true of virtual methods; you have to consider the consequences of bad overriders. And the same is true of protected methods; you have to consider the consequence of bad derived classes calling your code.

Designing robust code that has public methods is hard. Designing code that is robust and has virtual or protected methods is even harder; designing for extendability is a difficult problem in general, and shouldn’t be taken on lightly. *Consider sealing classes that aren’t designed for extension.*

