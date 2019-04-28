# Is C\# a strongly typed or a weakly typed language?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/15/2012 9:26:00 AM

-----

Presented as a dialogue, as is my wont\!

**Is C\# a strongly typed or a weakly typed language?**

Yes.

**That is unhelpful.**

I don't doubt it. Interestingly, if you rephrased the question as an "and" question, the answer would be the same.

**What? You mean, is C\# a strongly typed *and* a weakly typed language?**

Yes, C\# is a strongly typed language and a weakly typed language.

**I'm confused.**

Me too. Perhaps you should tell me precisely what you mean by "strongly typed" and "weakly typed".

**Um. I don't actually know what I mean by those terms, so perhaps that is the question I should be asking. What does it *really* mean for a language to be "weakly typed" or "strongly typed"?**

"Weakly typed" means "*this language uses a type verification system that I find distasteful*", and "strongly typed" means "*this language uses a type system that I find attractive*".

**No way\!**

Way, dude.

**Really?**

These terms are meaningless and you should avoid them. [Wikipedia](http://en.wikipedia.org/wiki/Strong_typing) lists *[eleven](https://www.youtube.com/watch?v=ll7rWiY5obI)* different meanings for "strongly typed", several of which contradict each other. Any time two people use "strongly typed" or "weakly typed" in a conversation about programming languages, odds are good that they have two subtly or grossly different meanings in their heads for those terms, and are therefore automatically talking past each other.

**But surely they mean *something* other than "unattractive" or "attractive"\!**

I do exaggerate somewhat for comedic effect. So lets say: a more-strongly-typed language is one that has some *restriction* in its type system that a more-weakly-typed language it is being compared to lacks. That's all you can really say without more context.

**How can I have sensible conversations about languages and their type systems then?**

You can provide the missing context. Instead of using "strongly typed" and "weakly typed", actually describe the restriction you mean. For example, C\# is *for the most part* a **statically typed** language, because the compiler determines facts about the types of every expression. C\# is *for the most part* a **type safe** language because it prevents values of one static type from being stored in variables of an incompatible type (and other similar type errors). And C\# is *for the most part* a **memory safe** language because it prevents accidental access to bad memory.

Thus, someone who thinks that "strongly typed" means "the language *encourages* static typing, type safety and memory safety *in the vast majority of normal programs*" would classify C\# as a "strongly typed" language. C\# is certainly more strongly typed than languages that do not have these restrictions in their type systems.

But here's the thing: because C\# is a pragmatic language there is a way to override all three of those safety systems. Cast operators and "dynamic" in C\# 4 override compile-time type checking and replace it with runtime type checking, and "unsafe" blocks allow you to turn off type safety and memory safety should you need to. Someone who thinks that "strongly typed" means "the language *absolutely positively guarantees* static typing, type safety and memory safety *under all circumstances*" would quite rightly classify C\# as "weakly typed". C\# is not as strongly typed as languages that do enforce these restrictions all the time.

So which is it, strong or weak? It is impossible to say because it depends on the point of view of the speaker, it depends on what they are comparing it to, and it depends on their attitude towards various language features. It's therefore best to simply avoid these terms altogether, and speak more precisely about type system features.

