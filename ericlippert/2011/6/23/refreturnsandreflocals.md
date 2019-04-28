# Ref returns and ref locals

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/23/2011 7:01:00 AM

-----

"Ref returns" are the subject of [another great question from StackOverflow](http://stackoverflow.com/questions/6339602/why-doesnt-c-support-the-return-of-references/6346059#6346059) that I thought I might share with a larger audience.

Ever since C\# 1.0 you've been able to create an "alias" to a variable by passing a "ref to a variable" to certain methods:

static void M(ref int x)  
{  
    x = 123;  
}  
...  
int y = 456;  
M(ref y);

Despite their different names, "x" and "y" are now aliases for each other; they both refer to the same storage location. When x is changed, y changes too because they are the same thing. Basically, "ref" parameters allow you to pass around variables as **variables** rather than as **values**. This is a sometimes-confusing feature (because it is easy to confuse "reference types" with "ref" aliases to variables,) but it is generally a pretty well-understood and frequently-used feature.

However, it is a little-known fact that the CLR type system supports additional usages of "ref", though C\# does not. The CLR type system also allows methods to return refs to variables, and allows local variables to be aliases for other variables. The CLR type system however does not allow for fields that are aliases to other variables. Similarly arrays may not contain managed references to other variables. Both fields and arrays containing refs are illegal because making it legal would overly complicates the garbage collection story. (I also note that the "managed reference to variable" types are not convertible to object, and therefore may not be used as type arguments to generic types or methods. For details, see the CLI specification Partition I Section 8.2.1.1, "Managed pointers and related types" for information about this feature.)

As you might expect, it is entirely possible to create a version of C\# which supports both these features. You could then do things like

static ref int Max(ref int x, ref int y)  
{  
  if (x \> y)  
    return ref x;  
  else  
    return ref y;  
}

Why do this? It is quite different than a conventional "Max" which returns the larger of two *values*. This returns the larger *variable* itself, which can then be modified:

int a = 123;  
int b = 456;  
ref int c = ref Max(ref a, ref b);  
c += 100;  
Console.WriteLine(b); // 556\!

Kinda neat\! This would also mean that ref-returning methods could be the left-hand side of an assignment -- we don't need the local "c":

int a = 123;  
int b = 456;  
Max(ref a, ref b) += 100;  
Console.WriteLine(b); // 556\!

Syntactically, 'ref' is a strong marker that something weird is going on. Every time the word "ref" appears before a variable usage, it means "I am now making some other thing an alias for this variable". Every time it appears before a declaration, it means "this thing must be initialized with an variable marked with ref".

I know empirically that it is possible to build a version of C\# that supports these features because I have done so in order to test-drive the possible feature. Advanced programmers (particularly people porting unmanaged C++ code) often ask us for more C++-like ability to do things with references without having to get out the big hammer of actually using pointers and pinning memory all over the place. By using managed references you get these benefits without paying the cost of screwing up your garbage collection performance.

We have considered this feature, and actually implemented enough of it to show to other internal teams to get their feedback. However at this time based on our research **we believe that the feature does not have broad enough appeal or compelling usage cases to make it into a real supported mainstream language feature**. We have other higher priorities and a limited amount of time and effort available, so we're not going to do this feature any time soon.

Also, doing it properly would require some changes to the CLR. Right now the CLR treats ref-returning methods as legal but unverifiable because we do not have a detector that detects and outlaws this situation:

static ref int M1(ref int x)  
{  
  return ref x;  
}  
  
static ref int M2()  
{  
  int y = 123;  
  return ref M1(ref y); // Trouble\!  
}  
static int M3()  
{  
    ref int z = ref M2();  
    return z;  
}

M3 returns the contents of M2's local variable, but the lifetime of that variable has ended\! It is possible to write a detector that determines uses of ref-returns that clearly do not violate stack safety. We could write such a detector, and if the detector could not prove that lifetime safety rules were met then we would not allow the usage of ref returns in that part of the program. It is not a huge amount of dev work to do so, but it is a lot of burden on the testing teams to make sure that we've really got all the cases. It's just another thing that increases the cost of the feature to the point where right now the benefits do not outweigh the costs.

**If we implemented this feature some day, would you use it? For what? Do you have a really good usage case that could not easily be done some other way?** If so, please leave a comment. The more information we have from real customers about why they want features like this, the more likely it will make it into the product someday. It's a cute little feature and I'd like to be able to get it to customers somehow if there is sufficient interest. However, we also know that "ref" parameters is one of the most misunderstood and confusing features, particularly for novice programmers, so we don't necessarily want to add more confusing features to the language unless they really pay their own way.

