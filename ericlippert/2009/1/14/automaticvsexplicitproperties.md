# Automatic vs Explicit Properties

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/14/2009 12:11:25 PM

-----

Here's a question I got from a C\# user last year, a question I get fairly frequently:

> User: With “regular” explicit properties, I tend to use the private backing field directly from within the class. Of course, with an automatic property, you can’t do this. My concern is that in the future, if I decide I need an explicit property for whatever reason, I’m left with the choice of changing the class implementation to use the new private field, or continue going through the property. I’m not sure what the right thing to do in this case is.

You say “for whatever reason”, and that's key. The answer to your question will depend **entirely** upon what the reason was that motivated the change. If the reason that motivated the change from automatically implemented property to explicitly implemented property was to **change the semantics** of the property then you should evaluate whether the desired semantics when accessing the property *from within the class* are identical to or different from the desired semantics when accessing the property *from outside the class*. If the result of that investigation is “from within the class, the desired semantics of accessing this property are *different* from the desired semantics of accessing the property from the outside”, then your edit has introduced a bug. **You should fix the bug**. If they are *the same*, then your edit has not introduced a bug; **keep the implementation the same**. That is a bit abstract. Let’s get more concrete. Suppose you have  

sealed class BankAccount  
{  
  public decimal Balance { get; set; }

and somewhere in the class you have a calculation:  

  if (Balance \> 0) …

Then one day you decide to change the semantics of Balance:  

sealed class BankAccount  
{  
  private decimal balance;  
  public decimal Balance  
  {  
    get  
    {  
      if (\!CurrentUser.HasReadAccess(this))  
        throw new SecurityException();  
      return balance;  
…

OK, you’ve changed the semantics of Balance. Now, everywhere inside your class, *do you intend all calculations that at present use Balance to **have** these security semantics, or do you intend them to **not have** these security semantics?* If the former, **keep the calling code the way it is; it is correct; changing it will introduce a bug.** If the latter, **change it; it is now wrong; changing it will fix a bug.** If the reason that you changed the property from automatic to explicit was NOT to modify the semantics then… then… then **why on Earth did you change it?** And why are you contemplating making *further* inessential changes? I presume that in this case there must be *some* motivation for changing correct working code; if you have some motivation that is causing you to change correct, working  code, I suppose it can be applied equally well to all the other correct, working code in the class that calls that property.

> User: Thanks. This is on my mind because I’m starting my first C\# 3.0 project, and am experimenting with the new features. The scenario that tripped me up is exactly as you state; someone making a future semantic change.

Then your question is actually an instance of a more general question: **“how do I future-proof a design?”** That is, how do I design and implement a class hierarchy now so that inevitable changes in the future are easier and less bug-prone?

That's a hard question, one which someone more knowledgeable than I am could write a whole book on. Next time on FAIC I'll give a few musings on this topic.

