# School Days, School Days

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/18/2004 7:20:00 PM

-----

Just a couple random notes today, following up on a few threads and reminiscing about school days. 

My old friend and erstwhile professor [Prabhakar Ragde](http://db.uwaterloo.ca/~plragde "http://db.uwaterloo.ca/~plragde") has some good [comments](/ericlippert/archive/2004/03/12/88731.aspx#91898 "http://blogs.msdn.com/ericlippert/archive/2004/03/12/88731.aspx#91898") and related links on my recent posting about [trends in Computer Science education in the United States](/ericlippert/archive/2004/03/12/88731.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/12/88731.aspx").  This is a subject which I know Professor Ragde has been interested in for a long time -- I recall reading a magazine article clipping on his office door in the Davis Center in, oh, must have been 1991, with the headline "**Study Shows Women Executives Interupt People As Often As Men**".  Funny what things stick in [one's](/ericlippert/archive/2003/12/02/53427.aspx "http://blogs.msdn.com/ericlippert/archive/2003/12/02/53427.aspx") memory.  

Anyway, Professor Ragde has at least three, probably more, ambitious projects underway.  First, I assume that he's still maintaining his position as the youngest curmudgeon on campus -- no mean feat given the massive influx of young talent in the CS department alone.  Second, it appears that he's attempting to dominate the classical music world by raising highly talented [children](http://db.uwaterloo.ca/~plragde/children.html "http://db.uwaterloo.ca/~plragde/children.html").  Just as ambitious, he's [redesigned the introductory computer science curriculum at Waterloo](http://www.student.cs.uwaterloo.ca/~cs135/alternate-sequence.html "http://www.student.cs.uwaterloo.ca/~cs135/alternate-sequence.html").  There are a lot of interesting ideas about functional languages as pedagogic tools in there.  Should be fun -- when I took the introductory (non-functional) CS134 course at Waterloo from Professor Ragde in 1991, it was a brand-new course then, and still had a lot of bugs to get worked out -- I recall getting photocopies of the proofreaders' galleys rather than a textbook, for instance.  I am very excited and interested to see where this goes.  

Introductory courses are hard to design in so many ways.  For instance, I noticed when I was taking CS134 that the distribution of marks in the class was bimodal -- there were a big chunk of people getting 95-100% and a big chunk barely passing.  Those people in the former group weren't really learning any new technical material, and those people in the latter were struggling to keep up.  I think for that reason alone it would be interesting to see how first year students handle functional languages; most high school students have little experience with Scheme, and so not only might the playing field be more level, but everyone might learn some CS fundamentals rather than coasting through easy, old-hat procedural programming. 

Introductory courses are also the place where a lot of people discover that they're really not interested in a subject and quit early.  That's not necessarily a bad thing -- I'm very glad that I learned early that though I find astrophysics fascinating, I'm really not very good at it.  The risk of course is that you end up accidentally getting rid of people who would add value and thrive in a particular profession; the trend lines for women entering computer science are particularly worrisome, as I've already noted.  Professor Ragde comments: 

I am currently designing a first-year course from scratch and would welcome comment on how it can be more effective at attracting and retaining women in the profession \[...\] But whatever I do will not be sufficient. We need a general cultural change. 

If you've got anything germane to add, comment here and I'll make sure that it gets passed on. 

Finally, while Professor Ragde was in the process of proposing his Scheme scheme, [a lot of people commented on the proposal](http://db.uwaterloo.ca/~plragde/scheme-comments.txt "http://db.uwaterloo.ca/~plragde/scheme-comments.txt").  Reading the comments was not only interesting for the varied positions and takes on the subject -- technical, political, pedagogical -- but also because it brought back a lot of fond memories. It's nice to see that many old friends and acquaintances are still active in University politics.   

As you can see from my comments (towards the bottom), I was thinking a lot about some of the issues that I've blogged about recently when I wrote them.  At the risk of being somewhat redundant, I'll reproduce them here. 

Your proposal is quite interesting.  I have often wondered why schools concentrate upon teaching principles of object oriented programming. 

The most obvious alleged benefit is that OOP languages such as C++, Java, C\# are in *widespread use in industry*.  But I don't really buy that argument.  Yes, UW is a highly *practical* school, but still surely the purpose of a UW CS degree is to study *theoretical* computer science.  

And really, most people in industry do not have a good grasp of theoretical OO principles.  More: they do not *need* a good grasp of OO principles except insofar as those principles serve their needs.  Heck, I design and implement programming languages for a living and it's not like we're in code reviews saying "*Well, Bob, this code works, but doesn't it violate the Liskov Substitution Principle*?"  

Which brings me to my second point -- why do we have OO principles in the first place?  Not because they are cool, I hope.  Rather, because OOP is a style of programming which emphasizes encapsulation, abstraction, contracts, information hiding, extension through inheritance, etc, etc, etc.  These are things which help in the design and implementation of *large scale software*.  

Super.  That's goodness for me -- I work on systems like that.  But in an introductory course in computer science, typically the students are working on small, simple programs by themselves.  *OO languages are not necessarily good pedagogically at the introductory level. *

What I sometimes see when I interview people and review code is symptoms of a disease I call **Object Happiness**.  Object Happy people feel the need to apply principles of OO design to small, trivial, throwaway projects.  They invest lots of unnecessary time making pure virtual abstract base classes -- writing programs where IFoos talk to IBars but there is only one implementation of each interface\!  I suspect that early exposure to OO design principles divorced from any practical context that motivates those principles leads to object happiness. People come away as OO True Believers rather than OO pragmatists. Hopefully the co-op program shocks them out of it, but better to not get Happy in the first place. 

So I agree with you that earlier exposure to functional programming is a good idea.  Not just from a theoretical standpoint, but also from a practical standpoint.  I strongly believe that as programming languages evolve we are going to see *an increasing number of functional language and declarative language features in mainstream industry production languages*.  (In the latest version of C\#, for example, there is improved support for delegates, moving towards a first-class-function model, and also some support for declarative attributes.  But both could be greatly improved further.) 

Of course, I might be somewhat biased.  I spent five of the last eight years working on JScript, which can be used as an imperative, object-oriented *and* functional language, and is usually found embedded inside declarative languages such as HTML, XML.  (My colleague at Netscape, Waldemar Horwat, once told me that Javascript was just another syntax for Common Lisp -- he was a pretty hard-core functional programmer.) 

Come to think of it, JScript might make a very interesting pedagogic language.  It's very easy to get productive right away by writing little imperative scripts, it implements functional language features such as closures and anonymous functions, and it has an interestingly non-standard approach to OOP (prototype inheritance).

