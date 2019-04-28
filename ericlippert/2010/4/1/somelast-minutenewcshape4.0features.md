# Some Last-Minute New C\# 4.0 Features

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/1/2010 6:26:00 AM

-----

As I’m sure you know by now, we are done implementing C\# 4. We’ve added support for interoperability with dynamic languages and legacy object models, named and optional parameters, the ability to “link” against interfaces from a Primary Interop Assembly, and my favourite feature, covariance and contravariance of interface and delegate types.

Now, sometimes we manage to find time in the schedule to fit in small additional features that do not directly align with the larger “theme” of the release, but are useful features in their own right. In C\# 3 we were all about LINQ features, but we did manage to also sneak in auto-properties and partial methods, neither of which directly supported our LINQ mission. But because they were clearly useful and relatively easy to design, implement, test and document, we got them in there.

Today I am announcing that we managed to get two additional operators into C\# 4 that we have not hitherto announced: the “goes to” operator , written --\> and the “is approached by” operator, written \<--. In fact, **we managed to get both of them into the last community preview, so if you have the CTP build, you can try them out today\!** (As you probably know, we do not like to ship features without running them by the community first.)

Here are examples of both operators:

 

int x = 10;  
// this is read "while x goes to zero"  
while (x --\> 0)  
{  
    Console.WriteLine("x = {0}", x);  
}

As you can see, this does exactly what you’d expect: x becomes 9, 8, 7, and so on, as it “goes to zero”. Our awesome compiler architect Neal heard via his contacts in the industry that the latest version of Java has this operator; wanting to maintain our massive lead in language features over Java, of course we worked overtime to deliver its benefits to our valued C\# developer community.

Shortly thereafter, our awesome language specification manager Mads realized that we could generalize this to the inverse operator, the “is approached by" operator. We therefore implemented it too:

 

int x = 10;  
// this is read "while zero is approached by x"  
while (0 \<-- x)  
{  
    Console.WriteLine("x = {0}", x);  
}

And as you can see again, x takes on each value as it approaches zero. The exact semantics of this operator are subtly different than the goes-to operator, but I’ll describe the differences in a later post.

Pretty neat, eh? I am sure you guys can think of all sorts of wonderful uses for these new operators. The time pressure was really hard for this one, but they came through.

Finally, I know we've said that we want to maintain language-feature parity between C\# and Visual Basic; unfortunately we were unable to get these operators into VB this time around. Perhaps in a future version.

UPDATE: Happy April Fool's Day everyone. For those of you who I fooled -- including at least one tester on the C\# QA team, ha ha ha \! -- of course **we are not announcing new operators the day before we ship C\# 4.0**.  "while (x --\> 0)" is the same as "while( x--   \>   0 )" which is of course perfectly legal and sensible code. This is an old joke; it works in any language with a "--" decrement operator.

