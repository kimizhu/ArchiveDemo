# Chaining simple assignments is not so simple

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/11/2010 6:59:00 AM

-----

UPDATE: I interrupt this episode of FAIC with a request from my friend and colleague Lucian, from the VB team, who wonders whether it is common in C\# to take advantage of the fact that assignment expressions are expressions. The most common usage of this pattern is the subject of this blog entry: the fact that "chained" assignment works at all is a consequence of the fact that assignments are expressions, not statements. There are other uses too; one could imagine something like "return this.myField = x;" as a short cut for "this.myField = x; return this.myField;" -- perhaps we are performing some computation and then recording the results for use later. Or perhaps we've got something like myNonNullableString = (myNullableString = Foo()) ?? "\<null\>"; -- there are any number of ways this idiom *could* be used.

I do not use this idiom myself; I'm of the opinion that side effects such as assignments are best represented by putting each in a statement of its own, rather than as something embedded in a larger expression. My question for you is: **do you use assignments as expressions?** If so, how and why? Note that I am looking for mundane, "real world" examples of this pattern, not clever ideas about how this could in theory be used. If you've got one, please leave it in the comments and I'll pass it along to Lucian. Thanks\!

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Today I examine another myth about C\#. Consider the following code:

 

a = b = c;

This is legal; you can make arbitrarily long chains of simple assignments. This pattern is most often seen in something like

 

int i, j, k;  
i = j = k = 123;

I often hear that this works “*because assignment is right-associative and results in the value of the right-hand side*”.

Well, that’s half true. It is right-associative; obviously this has to be equivalent to

 

i = (j = (k = 123)));

It doesn’t make any sense to parenthesize it from the left. Now, in this particular example, the statement is true, but in general it is not. The result of the simple assignment operator is **not** the value of the right hand side:

 

const int x = 10;  
short y;  
object z;  
z = y = x;  
System.Console.WriteLine(z.GetType().ToString());

This prints “System.Int16”, not “System.Int32”. The value of the right-hand side of “y = x” is clearly an int, but we do not assign a reference to a boxed int to z, we assign a reference to a boxed short\!

So then is the correct statement “*… results in the value of the left-hand side*”?

Nope, that’s not right either, and we can prove it.

 

class C  
{  
  private string x;  
  public string X {  
    get { return x ?? ""; }  
    set { x = value; } }  
  static void Main()  
  {  
    C c = new C();  
    object z;  
    z = c.X = null;  
    System.Console.WriteLine(z == null);  
    System.Console.WriteLine(c.X == null);  
  }  
}

This prints “True / False” – the result of the assignment operator is not the value of the left-hand-side. The value of the left hand side is the empty string but the value of the operator is null.

Heck, the left hand side need not even *have* a value. Write-only properties are weird and rare, but legal; if there were no getter then the left hand side c.X would not *have* a value\!

The correct statement should now be pretty easy to deduce: *the result of the simple assignment operator is the value that was assigned to the left-hand side*.

