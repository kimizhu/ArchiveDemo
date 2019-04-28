# Four switch oddities

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/13/2009 9:33:00 AM

-----

The C\# switch statement is a bit weird. Today, four quick takes on things you probably didn't know about the switch statement.

Case 1:

You probably know that it is illegal to "fall through" from one switch section to another:

 

switch(attitude)  
{  
  case Attitude.HighAndMighty:  
    Console.WriteLine("High");  
    // we want to fall through, but this is an error  
  case Attitude.JustMighty:  
    Console.WriteLine("Mighty");  
    break;  
}

But perhaps you did not know that it is *legal* to force a fall-through with a goto:

 

switch(attitude)  
{  
  case Attitude.HighAndMighty:  
    Console.WriteLine("High");  
    goto case Attitude.JustMighty;  
  case Attitude.JustMighty:  
    Console.WriteLine("Mighty");  
    break;  
}

Pretty neat\! Cases are semantically just labels to which the switch does a conditional branch; we let you do an explicit branch if you want to.

Case 2:

A common and confusing error that C programmers using C\# (like me\!) make all the time:

 

switch(attitude)  
{  
  case Attitude.HighAndMighty:  
    Console.WriteLine("High and Mighty");  
    break;      
  case Attitude.JustMighty:  
    Console.WriteLine("Just Mighty");  
    break;  
  default:  
    // Do nothing  
}

That's an error because in the default case, you "fall through". Admittedly, there is nothing to "fall through" to, but the compiler is picky about this one. It requires that *every* switch section, *including the last one*, have an *unreachable end point*. The purpose of this rule, and of the no-fall-through rule in general, is that we want you to be able to arbitrarily re-order your switch sections without accidentally introducing a breaking change. Fix it by making the "do nothing" explicit with a break statement.

This is particularly confusing because some people interpret the error message as saying that the problem is falling *into* the default case, when actually the problem is that you're falling *out of* the default case.

Case 3:

As I discussed earlier, a declaration space is a region of code in which two declared things may not have the same name. The foreach loop implicitly defines its own declaration space, so this is legal:

 

foreach(var blah in blahs) { ... }  
foreach(var blah in otherblahs) { ... }

Switch blocks also define their own declaration spaces, but switch sections do not:

 

switch(x)  
{  
  case OneWay:  
    int y = 123;  
    FindYou(ref y);  
    break;  
  case TheOther:  
    double y = 456.7; // illegal\!  
    GetchaGetcha(ref y);  
    break;  
}

You can solve this problem in a number of ways; the easiest is probably to wrap the body of the switch section in curly braces:

 

switch(x)  
{  
  case OneWay:  
  {  
    int y = 123;  
    FindYou(ref y);  
    break;  
  }  
  case TheOther:  
  {  
    double y = 456.7; // legal\!  
    GetchaGetcha(ref y);  
    break;  
  }  
}

which tells the compiler "no, *really*, I want these to be different declaration spaces".

If you have a variable that you want to declare once and use it in a bunch of different places, that's legal, but a bit strange:

 

switch(x)  
{  
  case OneDay:  
    string s;  
    SeeYa(out s);  
    break;  
  case NextWeek:  
    s = "hello"; // legal, we use the declaration above.  
    Meetcha(ref s);  
    break;  
  }  
}

That looks a bit weird, I agree, but it also looks a bit weird to have one switch block with two unbraced competing declarations in it. There are pros and cons of each, the design team had to pick one way or the other, and they chose to have switch cases not define a declaration space.

Case 4:

A funny consequence of the "reachability analysis" rules in the spec is that this program fragment is not legal:

 

int M(bool b)  
{  
  switch(b)  
  {  
    case true: return 1;  
    case false: return 0;  
  }  
}

Of course in reality you would probably write this as the far more concise "return b ? 1 : 0;" but shouldn't that program be *legal*? It is not, because the reachability analyzer reasons as follows: since there is no "default" case, the switch might choose some option that is not either case. Therefore the end point of the switch is reachable, and therefore we have an int-returning method with a reachable code path that falls off the end of the method without returning. 

Yeah, the reachability analyzer is not very smart. It does not realize that there are *only two possible control flows* and that we've covered all of them with returns. And of course, if you switch on a byte and have cases for each of the 256 possibilities, again, we do not detect that the switch is exhaustive.

This shortcoming of the language design is silly, but frankly, we have higher priorities than fixing this silly case. If you find yourself in this unfortunate case, just stick a "default:" label onto one of the sections and you'll be fine.

\-- Eric is on vacation; this posting was prerecorded. --

