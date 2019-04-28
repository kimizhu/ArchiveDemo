# Why Can't I Access A Protected Member From A Derived Class, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/24/2008 12:46:48 PM

-----

Holy goodness, I've been busy. The MVP Summit was fabulous for us; thanks to all who attended and gave us great feedback on our ideas for evolving the languages and tools. And I've been doing some longer-lead thinking and working on language futures, which I will blog about at a later date.

[Last time](http://blogs.msdn.com/ericlippert/archive/2008/03/28/why-can-t-i-access-a-protected-member-from-a-derived-class-part-two-why-can-i.aspx) I promised another oddity of "protected" semantics. An issue that I get asked about all the time involves the meaning of this code:

 

namespace Foo {  
  public class Base {  
    protected internal void M() { ... }  
  }  
}

Many people believe that this means " M is accessible to all derived classes that are in this assembly." It does not. It actually means "M is accessible to all derived classes **and** to all classes in this assembly".  That is, it is the *less restrictive* combination, not the *more restrictive* combination.

This is counterintuitive to a lot of people. I have been trying to figure out why, and I think I've got it. I think people conceive of internal, protected and private as *restrictions* from the "natural" state of public. With that model, protected internal means "apply both the protected restriction and the internal restriction".

That's the wrong way to think about it. Rather, internal, protected and public are weakenings of the "natural" state of private. private is the default in C\#; if you want to make something have broader accessibility, you've got to say so. With that model, then it becomes clear that protected internal is a weaker restriction than either alone.

(As an aside: An irksome fact about the design of C\# is that we do not *consistently* default to the more restricted mode. I like that private is the default visibility for most members and "instance", not "virtual" is the default method behaviour. But why aren't classes sealed by default? This would emphasize that participation in an inheritance hierarchy needs to be part of the design of a class.)

We get the feature request fairly frequently to add the "more restricted" form, but the thing is, I'm not sure what *use* it actually is. If the member is already marked as internal then *the only people who can use it are your coworkers*. What is the harm in allowing them to party on an internal member from outside the class hierarchy?

Of course, if we did add the feature, the codegen would be trivial; this kind of accessibility is already supported natively in the CLR. The work would come in defining a sensible syntax for it. protected with internal or protected and internal might work. Or, we could define a new keyword having this meaning. proternal, for example. Or intected. (The former sounds very positive; the latter, like bad news from a dentist.)

Long story short, I would not expect this feature any time soon.

Next time, some thoughts on your comments to my last entry.

