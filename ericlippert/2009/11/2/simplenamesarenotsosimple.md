# Simple names are not so simple

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/2/2009 6:23:00 AM

-----

C\# has many rules that are designed to prevent some common sources of bugs and encourage good programming practices. So many, in fact, that it is often quite confusing to sort out exactly which rule has been violated. I thought I might spend some time talking about what the different rules are. We'll finish up with a puzzle.

To begin with, it will be vital to understand [the difference between *scope* and *declaration space*](http://blogs.msdn.com/ericlippert/archive/2009/08/03/what-s-the-difference-part-two-scope-vs-declaration-space-vs-lifetime.aspx). To refresh your memory of my earlier article: the *scope* of an entity is *the region of text in which that entity may be referred to by its unqualified name*. A *declaration space* is a region of text in which *no two things may have the same name* (with an exception for methods which differ by signature.) A "local variable declaration space" is a particular kind of declaration space used for declaring local variables; local variable declaration spaces have special rules for determining when they overlap.

The next thing that you have to understand to make any sense o this is what a "simple name" is. A *simple name* is always either just a plain identifier, like "x", or, in some cases, a plain identifier followed by a type argument list, like "Frob\<int, string\>".

Lots of things are treated as "simple names" by the compiler: local variable declarations, lambda parameters, and so on, always have the first form of simple name in their declarations. When you say "Console.WriteLine(x);" the "Console" and "x" are simple names but the "WriteLine" is not. Confusingly, there are some textual entities which have the form of simple names, but are not treated as simple names by the compiler. We might talk about some of those situations in later fabulous adventures.

So, without further ado, here are some relevant rules which are frequently confused. It's rules 3 and 4 that people find particularly confusing.

1\) It is illegal to refer to a local variable before its declaration. (This seems reasonable I hope.)  
2\) It is illegal to have two local variables of the same name in the same local variable declaration space or nested local variable declaration spaces.  
3\) Local variables are in scope throughout the entire block in which the declaration occurs. This is in contrast with C++, in which local variables are in scope in their block only at points after the declaration.  
4\) For every occurrence of a simple name, whether in a declaration or as part of an expression, all uses of that simple name within the immediately enclosing local variable declaration space must refer to the same entity.

The purpose of all of these rules is to prevent the class of bugs in which the reader/maintainer of the code is tricked into believing they are referring to one entity with a simple name, but are in fact accidentally referring to another entity entirely. These rules are in particular designed to prevent nasty surprises when performing what ought to be safe refactorings.

Consider a world in which we did not have rules 3 and 4. In that world, this code would be legal:

 

class C  
{  
  int x;  
  void M()  
  {  
    // 100 lines of code  
    x = 20; // means "this.x";  
    Console.WriteLine(x); // means "this.x"  
    // 100 lines of code  
    int x = 10;  
    Console.WriteLine(x); // means "local x"  
  }  
}

This is hard on the person reading the code, who has a reasonable expectation that the two "Console.WriteLine(x)" lines do in fact both print out the contents of the same variable. But it is particularly nasty for the maintenance programmer who wishes to impose a reasonable coding standard upon this body of code. "Local variables are declared at the top of the block where they're used" is a reasonable coding standard in a lot of shops. But changing the code to:

 

class C  
{  
  int x;  
  void M()  
  {  
    int x;  
    // 100 lines of code  
    x = 20; // no longer means "this.x";  
    Console.WriteLine(x); // no longer means "this.x"  
    // 100 lines of code  
    x = 10;  
    Console.WriteLine(x); // means "local x"  
  }  
}

changes the meaning of the code\! We wish to discourage authoring of multi-hundred-line methods, but making it harder and more error-prone to refactor them into something cleaner is not a good way to achieve that goal.

Notice that the original version of this program, rule 3 means that the program violates rule 1 -- the first usage of "x" is treated as a reference to the local before it is declared. The fact that it violates rule 1 because of rule 3 is precisely what prevents it from being a violation of rule 4\! The meaning of "x" is consistent throughout the block; it always means the local, and therefore is sometimes used before it is declared. If we scrapped rule 3 then this would be a violation of rule 4, because we would then have two inconsistent meanings for the simple name "x" within one block.

Now, these rules do not mean that you can refactor willy-nilly. We can still construct situations in which similar refactorings fail. For example:

 

class C  
{  
  int x;  
  void M()  
  {  
    {  
      // 100 lines of code  
      x = 20; // means "this.x";  
      Console.WriteLine(x); // means "this.x"  
    }  
    {  
      // 100 lines of code  
      int x = 10;  
      Console.WriteLine(x); // means "local x"  
    }  
  }  
}

This is perfectly legal. We have the same simple name being used two different ways in two different blocks, but the immediately enclosing block of each usage does not overlap that of any other usage. The local variable is in scope throughout its immediately enclosing block, but that block does not overlap the block above. In this case, it is safe to refactor the declaration of the local to the top of its block, but not safe to refactor the declaration to the top of the outermost block; that would change the meaning of "x" in the first block. Moving a declaration up is almost always a safe thing to do; moving it out is not necessarily safe.

Now that you know all that, here's a puzzle for you, a puzzle that I got completely wrong the first time I saw it:

 

using System.Linq;  
class Program  
{  
  static void Main()  
  {  
    int\[\] data = { 1, 2, 3, 1, 2, 1 };  
    foreach (var m in from m in data orderby m select m)  
      System.Console.Write(m);  
  }  
}

It certainly looks like name "m" is being used multiple times to mean different things. Is this program legal? If yes, why do the rules for not re-using simple names not apply? If no, precisely what rule has been violated? 

\[Eric is on vacation; this posting was pre-recorded.\]

