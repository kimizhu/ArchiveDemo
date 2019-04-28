# Knights, Knaves, Protected and Internal

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/25/2010 6:50:00 AM

-----

[![Knight](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/KnightsKnavesProtectedandInternal_EB1D/Knight_thumb.png "Knight")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/KnightsKnavesProtectedandInternal_EB1D/Knight_2.png) When you override a virtual method in C\# you are required to ensure that the stated accessibility of the overridden method - that is, whether it is public, internal, protected or protected internal(\*) – is exactly re-stated in the overriding method. Except in one case. I refer you to section 10.6.4 of the specification, which states:

> an override declaration cannot change the accessibility of the virtual method. However, if the overridden base method is *protected internal* and it is declared in a different assembly than the assembly containing the override method then the override method’s declared accessibility must be *protected*.

What the heck is up with that? Surely if an overridden method is *protected internal* then it only makes sense that the overriding method should be exactly the same: *protected internal*.

I’ll explain why we have this rule, but first, a brief digression.

A certain island is inhabited by only knights and knaves. Knights make only true statements and only answer questions truthfully; knaves make only false statements and only answer questions untruthfully. If you walk up to an inhabitant of the (aptly-named) Island of Knights and Knaves you can rapidly ascertain whether a particular individual is a knight or a knave by asking a question you know the answer to. For example “does two plus two equal four?” A knight will answer “yes” (\*\*), and a knave will answer “no”. Knaves are prone to saying things like “my mother is a male knight”, which is plainly false.

It might seem at first glance that there is no statement which could be made by both a knight and a knave. Since knights tell the truth and knaves lie, they cannot both make the same statement, right? But in fact there are many statements that can be made by both. Can you think of one?

.

.

.

.

.

.

.

Both a knight and a knave can say “I am a knight.”

How does that work? The reason this works is because the pronoun “I” refers to different people when uttered by different people. If Alice, a knight, makes the statement “I am a knight”, she is asserting the truth that “Alice is a knight”. If Bob, a knave, makes the statement “I am a knight”, he is not asserting the true statement “Alice is a knight” but rather the false statement “Bob is a knight”. Similarly, both Alice and Bob can assert the statement "My name is Alice," for the same reason.

And that’s why overriding methods in a different assembly aren’t “protected internal”. The modifier “internal” is like a pronoun; it refers to the current assembly. When used in two different assemblies it means different things. The purpose of the rule is to ensure that a derived class does not make the accessibility domain of the virtual member any larger or smaller.

An analogy might help. Suppose a protected resource is a car, an assembly is a dwelling, a person is a class and descendent is a derived class.

  - Alice has a Mazda and lives in House with her good friend Charlie.
  - Charlie has a child, Diana, who lives in Apartment.
  - Alice has a child, Elroy, who lives in Condo with his good friend Greg.
  - Elroy has a child – Alice’s grandchild -- Frank, who lives in Yurt.

Alice grants access to Mazda to anyone living in House and any descendent of Alice. The people who can access Mazda are Alice, Charlie, Elroy, and Frank.

Diana does not get access to Mazda because she is not Alice, not a descendent of Alice, and not a resident of House. That she is a child of Alice’s housemate is irrelevant.

Greg does not get access to Mazda for the same reason: he is not Alice, not a descendent of Alice, and not a resident of House. That he is a housemate of a descendent of Alice is irrelevant. 

Now we come to the crux of the matter. **Elroy is not allowed to extend his access to Mazda to Greg.** Alice owns that Mazda and she said "myself, my descendents and my housemates". Her children don't have the right to extend the accessibility of Mazda beyond what she initially set up. Nor may Elroy deny access to Frank; as a descendent of Alice, Frank has a right to borrow the car and Greg cannot stop him by making it "private".

When Elroy describes what access he has to Mazda he is only allowed to say "I grant access to this to myself and my descendents" because that is what Alice already allowed. He cannot say "I grant access to Mazda to myself, my descendents and to the other residents of Condo".

\-----------------

(\*) Private virtual methods are illegal in C\#, which irks me to no end. I would totally use that feature if we had it.

(\*\*) Assuming that they answer at all, of course; there could be mute knights and knaves.

