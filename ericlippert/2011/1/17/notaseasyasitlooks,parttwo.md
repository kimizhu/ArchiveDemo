# Not as easy as it looks, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/17/2011 11:25:00 AM

-----

Holy goodness, did you guys ever find a lot of additional ways in which an "eliminate variable" refactoring can go wrong. Just a few of your observations: (again, in every case, "x" is eliminated.)

Any situation in which x is being **treated as a variable rather than a value** will pose a problem. Some obvious ones, like if x is on the left side of an assignment, or the target of a ++, or used as a "ref" or "out" parameter are straightforward. But there are some situations in which it is a bit surprising where x is being used as a variable. Suppose S is a mutable struct with field F:

 

S x = new S();  
x.F = 123;

Here we cannot simply say (new S()).F = 123 because new S() is not a variable. The mutable field requires a storage location to mutate but we don't have a storage location here.

A number of people pointed out that **[the C\# rules about how simple names may be used](http://blogs.msdn.com/b/ericlippert/archive/tags/simple+names/)** potentially pose problems:

 

int y = 2;  
void M()  
{  
  Action x = () =\> { Console.WriteLine(y); };  // refers to this.y  
  {  
    string y = "hello";  
    x();  
  }

The two inconsistent uses of the simple name y are legal here because the declaration spaces in which both are first used do not overlap. But the refactoring causes them to overlap; this needs to be refactored to

 

void M()  
{  
  {  
    string y = "hello";  
    ((Action)(()=\>{Console.WriteLine(this.y);})))();  
  }

It gets even worse if the inconsistent simple names cannot be fully qualified:

 

Action x = () =\> {int z = 123;};  
{  
  string z = "hello";  
  x();  
}

Now one of the local zeds must be renamed if the refactoring is to succeed.

You can also get up to mischief with anonymous types:

 

int x = 42;  
var a = new { x };

This must be refactored to

 

var a = new { x = 42 };

Similarly,

 

int x = k.y( );  
var a = new { x };

cannot be refactored to  

var a = new { k.y() };

because that changes the name of the anonymous type property. Moving an expression around can also muck with the definite assignment rules; this would have to be illegal to refactor:  

void Foo(out int b)  
{  
   int x = b = R();  
   if (Q()) return;  
   doSomething(x);  
}

because it becomes

 

void Foo(out int b)  
{  
   if (Q()) return;  
   doSomething(b = R());  
}

And now we have a method that returns without assigning the out parameter.

I already mentioned that moving an expression around can change the semantics of a program because the order of observed side effects can change. A number of horrid ways in which methods become dependent on side effects were noted, the nastiest of which was:

 

int x = LocksFoo();  
lock (Bar) { return M(x); }

The refactoring changes the order in which Foo and Bar are locked, and therefore might cause a deadlock if other code depends on Foo always being locked before Bar. Array initializers are a kind of expression that is only legal in a small number of situations; the refactoring has to take that into account:  

int\[\] x = {0,1};  
Console.WriteLine(x);

needs to be refactored into  

Console.WriteLine(new int\[\] {0, 1});

And finally, moving an expression around can move it from a checked to an unchecked context, or vice versa:  

int zero = 0;  
int one = 1;  
int x = int.MaxValue + one;  
Console.WriteLine(checked(x + zero));

The naive refactoring turns a program which succeeds into one which fails at runtime:

 

Console.WriteLine(checked(int.MaxValue + one + zero));

To be correct, the refactoring would have to introduce an unchecked block around the first addition\!

Thanks all, that was quite amusing.

