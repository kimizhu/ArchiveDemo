# Teach Yourself C\# In... how long?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/13/2010 6:39:00 AM

-----

![](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/2330.Book.jpg)

Earlier this year I was the technical editor of "Teach Yourself Visual C\# 2010 in 24 Hours" by Scott Dorman, and I am pleased to announce that it is available in stores now. Scott has pretty much completely rewritten this from the previous edition to ensure that it is totally up-to-date for C\# 4. If you're looking for a tutorial book aimed at beginners, this is a good one. (I am not super thrilled with the title; I've always agreed with Peter Norvig that this series really should be called "[Teach Yourself Programming In Ten Thousand Hours Because Realistically That Is About How Long It Takes To Gain Expertise At Any Skill](http://norvig.com/21-days.html)", but, whatever.)

Scott and the production team were kind enough to ask me to write the foreword, which I reproduce below. For more thoughts on why I enjoyed this book, read on:

\*\*\*\*\*

Over a decade ago, a small team of designers met in a small conference room on the second floor of Building 41 at Microsoft to create a brand-new language, C\#. The guiding principles of the language emphasized simplicity, familiarity, safety and practicality. Of course, all those principles needed to balance against one another; none are absolutes. The designers wanted the language to be simple to understand but not simplistic, familiar to C++ and Java programmers but not a slavish copy of either, safe by default but not too restrictive, and practical but never abandoning a disciplined, consistent, and theoretically valid design.

After many, many months of thought, design, development, testing and documentation, C\# 1.0 was delivered to the public. It was a pretty straightforward object-oriented language. Many aspects of its design were carefully chosen to ensure that objects could be organized into independently versionable components, but the fundamental concepts of the language came from ideas developed in object-oriented and procedural languages going back to the 1970’s or earlier.

The design team continued to meet three times a week in that same second floor conference room, to build upon the solid base established by C\# 1.0. By working with colleagues in Microsoft Research Cambridge and the CLR team across the street, the type system was extended to support parametric polymorphism on generic types and methods. They also added “iterator blocks” (sometimes known as “generators” in other languages) to make it easier to build iterable collections, and anonymous methods. Generics and generators had been pioneered by earlier languages such as CLU and Ada in the 1970’s and 1980’s; the idea of embedding anonymous methods in an existing method goes all the way back to the foundations of modern computer science in the 1950’s.

C\# 2.0 was a huge step up from its predecessor, but still the design team was not content. They continued to meet in that same second floor conference room three times a week. This time, they were thinking about fundamentals. Traditional “procedural” programming languages do a good job of basic arithmetic, but the problems faced by modern developers go beyond adding a column of numbers to find the average. They realized that programmers manipulate data by combining relatively simple operations in complex ways. Operations typically include sorting, filtering, grouping, joining and projecting collections of data. The concept of a syntactic pattern for “query comprehensions” that concisely describes these operations was originally developed in functional languages such as Haskell but also works well in a more imperative language like C\#. And thus LINQ – Language Integrated Query – was born.

After ten years of meeting for six hours a week in the same conference room, the need to teleconference with offsite team members motivated a change of venue to the fifth floor. The design team looked back on the last ten years to see what real-world problems were not solved well by the language, where there were “rough edges”, and so on. The increasing need to interoperate with both modern dynamic languages and legacy object models motivated the design of new language features like the “dynamic” type in C\# 4.0.

I figured it might be a good idea to do a quick look at the evolution of the C\# language here, in the foreword, because this is certainly not the approach taken in this book. And that is a good thing\! Authors of books for novices often choose to order the material in the order they learned it, which, as often as not, is the order in which the features were added to the language. What I particularly like about this book is that Scott chooses a sensible order to develop each concept, moving from the most basic arithmetical computations up to quite complex interrelated parts. Furthermore, his examples are actually realistic and motivating while still being clear enough and simple enough to be described in just a few paragraphs.

I’ve concentrated here on the evolution of the language, but of course the evolution of one language is far from the whole story. The language is just the tool you use to access the power of the runtime and the framework libraries; they are large and complex topics in of themselves. Another thing I like about this book is that it does not concentrate narrowly on the language, but rather builds upon the language concepts taught early on to explain how to make use of the power afforded by the most frequently used base class library types.

As my brief sketch of the history of the language shows, there’s a lot to learn here, even looking at just the language itself. I’ve been a user of C\# for ten years, and one of its designers for five, and I’m still finding out new facts about the language and learning new programming techniques every day. Hopefully your first twenty-four hours of C\# programming described in this book will lead to your own decade of practical programming and continual learning. As for the design team, we’re still meeting six hours a week, trying to figure out what comes next; I’m looking forward to finding out.  
  
Eric Lippert  
Seattle, Washington  
March 2010

