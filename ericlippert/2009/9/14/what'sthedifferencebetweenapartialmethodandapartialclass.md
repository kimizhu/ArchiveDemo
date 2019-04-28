# What's the difference between a partial method and a partial class?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/14/2009 11:16:00 AM

-----

Like "fixed" and "into", "partial" is also used in two confusingly similar-yet-different ways in C\#.

The purpose of a partial class is to allow you to textually break up a class declaration into multiple parts, usually parts found in separate files. The motivation for this feature was machine-generated code that is to be extended by the user by adding to it directly. When you draw a form in the forms designer, the designer generates a class for you representing that form. You can then further customize that class by adding more code to it. If you can edit the machine-generated code then any number of problems arise. What if it is re-generated? What if the machine is using the code itself as a persisted state for design-time information about the class, and your edit messes up the machine's parser? It's much better to simply put the machine-generated half in its own file, generate a comment that says "can't touch this", and put the user code in its own location.

There are other uses of partial classes that do not involve machine-generated code but they are relatively rare. Some cases where I see partial classes used are:

  - If a class is really large and implements a bunch of interfaces, sometimes one interface implementation per file makes sense. More often though this is a bad code smell which indicates a class that is trying to do too much; it might be better to split this thing up into multiple classes.
  - It's somtimes nice to put nested classes in their own files; making the containing class partial is the only good way to do that.
  - And so on.

Partial methods are a subtly different story. Like partial classes, partial methods are about combining multiple declarations of the same method to make machine-generated code scenarios better. But though the high-level purpose of the feature is the same, the details are rather different.

The way a partial method works is there are two declarations, the "latent" declaration and the "actual" declaration. The latent declaration does not have a body, like it was an abstract method. The actual declaration does. The latent declaration goes in the machine-generated side of a partial class, the actual declaration goes in the human-authored side. If there is an actual declaration, then the latent declaration is completely ignored. But if there is no actual declaration then all calls to the method are removed, just as it the method were compiled with the conditional attribute\! And in fact, the latent declaration is also logically removed when we spit out the metadata for the generated class; it's as if it never was.

The reason for this behaviour is because we wanted to enable this scenario:

 

 // Machine generated code:  
partial class MyFoo  
{  
  void ButtonClickEventHandler(/\*whatever\*/)  
  {  
      // call user code to see if they want to do anything at this time  
      OnBeforeButtonClick(whatever);  
      blah blah blah  
      // call user code again  
      OnAfterButtonClick(whatever);  
  }  
  partial void OnBeforeButtonClick(/\*whatever\*/);   
  partial void OnAfterButtonClick(/\*whatever\*/);  
...  

The user is going to be customizing the partial class; by putting in partial methods, the machine-generated side can create simple customization points all over the show that the user can then implement. But consider the down sides of this in a world without partial methods. The user is forced to implement empty methods. If they do not, they get errors. There are potentially hundreds of these empty methods to implement, which is vexing. And each of those methods generates a non-trivial amount of metadata, making the final binary larger than it needs to be. Disk space is cheap but network latency has replaced disk space as the factor that discourages large assemblies. If we can eliminate that metadata burden, that would be great.

Partial methods fit the bill. The user can provide actual implementations for as many methods as they choose, and now this becomes a "pay to play" model; you get as much implementation expense as the number of actual methods you implement.

The name of the feature during the design process was "latent and actual methods"; we strongly considered adding new keywords "latent" and "actual". But since the feature only makes sense in partial class scenarios, we decided to re-use the existing contextual keyword "partial" and renamed the feature. Hopefully the consonance between the two uses of "partial" helps more than the subtle differences hurt.

**SUPER EXTRA BONUS: Yet more partiality**

We considered adding a third kind of "partiality" to C\# 4; this feature made it through the design phase but got cut before implementation. (If there is high demand, we'll consider adding it to hypothetical future versions of C\#.) 

Sometimes you're in a machine-generated code scenario like this:

 

// Machine generated  
partial class C  
{  
  int blah;  
...

and then in the user-generated side of things, you want to do something like:

 

// User generated  
partial class C : ISerializable  
{  

And oh heck, I need to put the NotSerialized attribute on blah, but I cannot edit the text of blah because when it gets re-generated, that will be lost.

The idea of this new kind of partiality is to re-state the declaration of a member -- a field, method, nested type, whatever -- with metadata attributes. It's like a latent/actual method but in reverse; the "actual" thing is in the machine-generated side, the "latent" thing in the user-generated side is just there to add metadata:

 

\[NotSerialized\] **partial** int blah; // not actually a declaration of a field

I like this feature but during the design process I pushed back hard on using the "partial" keyword to have a third meaning subtly different from the other two. Adding this confusion once seemed justifiable, but twice? That's pushing it. We therefore settled on adding another contextual keyword:

 

\[NotSerialized\] **existing** int blah; // not actually a declaration of a field

Decorating a declaration with "existing" would mean "this is not a real declaration, this is a mention of an existing declaration; please verify that such a declaration exists somewhere else in this partial class, and add this metadata to that member".

Like I said, this handy feature got cut because of resource constraints. If you have really awesome scenarios that this would make easier, I'd love to hear about them; obviously I cannot make any promises about possible future features of unannounced, entirely hypothetical products. But real user scenarios are a highly motivating factor in getting budget for features.

