# The Root Of All Evil, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/28/2006 2:18:00 PM

-----

People often quote Knuth's famous statement that premature optimization is the root of all evil. Boy, has that ever been the theme of my life these last few weeks as I've been banging my head against the compiler trying to figure out how we're going to make it work with LINQ without breaking backwards compatibility. Some seemingly harmless premature optimizations in the compiler are eating up precious time dealing with how to work around them.

Before I can explain what exactly the premature optimization is that's drivin' me nuts, a short quiz. Suppose you've got an enumerated type E with value X.

 E e1 = E.X;  
E e2 = 0 | E.X;  
E e3 = E.X | 0;  
E e4 = E.X | 0 | 0;  
E e5 = 0 | E.X | 0;  
E e6 = 0 | 0 | E.X;  

All of these are legal according to the compiler. One of these is illegal according to the specification. Before you read on, take a guess as to which one, and why.

Now, producing sensible behaviour when we should be producing an error is the least bad of all possible spec violations, but still, it's irksome. It means that we have to ensure that we never comply with the spec, because if we suddenly turn this into an error, we might break an existing program.

Which of the above should be illegal? The first one is obviously good. The second and third are legal because section 13.1.3 says that literal zero is implicitly convertible to any enum, and section 14.10.2 says that you can binary-or two enumerated type values. We'll therefore convert the zero to the enumerated type and use the enum binary-or operator.

Since binary-or groups on the left, the fourth and fifth are legal. The left-hand part of the expression is evaluated first, and the right-hand zero is converted to the enumerated type.

The sixth case is illegal according to the spec, because this is the same as (0|0)|E.X – (0|0) is not a literal zero, it's a compile-time-constant expression that happens to equal zero, and there's no implicit conversion from that thing to any enumerated type\! And there's the premature optimization. After we've got a complete parse tree we walk through the parse tree making sure that all the types work out. Unfortunately the initial type binding pass bizarrely enough does arithmetic optimizations. It detects the 0|something and aggressively replaces it with just something , so as far as the compiler is concerned, the sixth case is the same as the second, which is legal. Argh\! This kind of optimization is way, way premature. It's the kind of thing you want to do very late in the game, possibly even during the code generation stage. Someone tried to do too much in one pass – the type annotation stage is no place for arithmetic optimizations, particularly when they screw up your type analysis in subtle ways like this\!

Next time I'll talk about how these same premature optimizations screw up definite assignment checking. There is a much related situation where there is a variable which the compiler treats as definitely assigned, and it IS definitely assigned, but the definite assignment specification requires us to produce an error. Bonus points to anyone who can suss it out from that vague description\!

