# Every public change is a breaking change

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/9/2012 9:08:00 AM

-----

Here's an inconvenient truth: just about every "public surface area" change you make to your code is a potential breaking change.

First off, I should clarify what I mean by a "breaking change" for the purposes of this article. If you provide a component to a third party, then a "breaking change" is a change such that the third party's code compiled correctly with the previous version, but the change causes a recompilation to fail. (A more strict definition would be that a breaking change is one where the code recompiles successfully but has a different meaning; for today let's just consider actual "build breaks".) A "potential" breaking change is a change which might cause a break, if the third party happens to have consumed your component in a particular way. By a "public surface area" change, I mean a change to the "public metadata" surface of a component, like adding a new method, rather than changing the behaviour of an existing method by editing its body. (Such a change would typically cause a difference in runtime behaviour, rather than a build break.)

Some public surface area breaking changes are obvious: making a public method into a private method, sealing an unsealed class, and so on. Third-party code that called the method, or extended the class, will break. But a lot of changes seem a lot safer; adding a new public method, for example, or making a read-only property into a read-write property. As it turns out, almost any change you make to the public surface area of a component is a potential breaking change. Let's look at some examples. Suppose you add a new overload:

// old component code:  
public interface IFoo {...}  
public interface IBar { ... }  
public class Component  
{  
    public void M(IFoo x) {...}  
}

Suppose you then later add

    public void M(IBar x) {...}

to Component. Suppose the consumer code is:

// consumer code:  
class Consumer : IFoo, IBar  
{  
   ...  
   component.M(this);  
   ...  
}

The consumer code compiles successfully against the original component, but recompiling it with the new component suddenly the build breaks with an overload resolution ambiguity error. Oops.

What about adding an entirely new method?

// old component code:  
...  
public class Component  
{  
  public void MFoo(IFoo x) {...}  
}

and now you add

public void MBar(IBar x) {...}

No problem now, right? The consumer could not possibly have been consuming MBar. Surely adding it could not be a build break on the consumer, right?

class Consumer  
{  
    class Blah  
    {  
        public void MBar(IBar x) {}  
    }  
    static void N(Action\<Blah\> a) {}  
    static void N(Action\<Component\> a) {}  
    static void D(IBar bar)  
    {  
        N(x=\>{ x.MBar(bar); });  
    }  
}

Oh, the pain. In the original version, overload resolution has two overloads of N to choose from. The lambda is not convertible to Action\<Component\> because typing formal parameter x as Component causes the body of the lambda to have an error. That overload is therefore discarded. The remaining overload is the sole applicable candidate; its body binds without error with x typed as Blah.

In the new version of Component the body of the lambda does not have an error; therefore overload resolution has two candidates to choose from and neither is better than the other; this produces an ambiguity error.

This particular "flavour" of breaking change is an odd one in that it makes almost every possible change to the surface area of a type into a potential breaking change, while at the same time being such an obviously contrived and unlikely scenario that no "real world" developers are likely to run into it. When we are evaluating the impact of potential breaking changes on our customers, we now explicitly discount this flavour of breaking change as so unlikely as to be unimportant. Still, I think its important to make that decision with eyes open, rather than being unaware of the problem.

