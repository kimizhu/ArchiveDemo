<div id="page">

# Aargh\! Part Seven

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/4/2004 10:17:00 AM

-----

<div id="content">

Q: How do pirates keep their socks from falling down? A: Thumbtacks. I am insanely busy with bug fixing and performance testing today, so once more I'll dip into my endless archive of rants about irksome coding practices I've seen one time too many. Gripe \#8: Assert the truth, the whole truth, and nothing but the truth Debug assertions should always describe **invariants** -- conditions that should *always* happen *no matter what*. They both help document your program, by making your invariants clear, and help catch bugs by alerting you when those invariants are violated. But assertions must describe what you believe to be **always** true about your algorithm, not what you **hope** is true: pv = malloc(10);  
Assert(NULL \!= pv); Aargh\! That's drivin' me nuts\! NULL is a legal return value of malloc and you therefore can't assert that malloc didn't return it. That condition, rare though it might be, has to be tested like any other. You can't simply declare that the world is going to turn out the way you'd like it. Sometimes you want to pop up warnings in your debug build when incredibly rare things happen. That's a great idea -- but create a little function called CreateDebugWarning or some such thing, so that it does not get confused with Assert. Gripe \#9: Don't be so darn friendly In C++ you can hide implementation details with the private keyword, but sometimes you want to have two classes which have the ability to party on each other's internals. For example, you might have a collection class and an enumerator class that need to have a private way to communicate with each other. Such classes are called "friendly classes". One day I grepped the headers of an unnamed Microsoft application for the word friend. It appeared over 600 times\! Aargh\! This is a bad sign. One class had *eleven* different friend classes. When you have that many friends, there really is no difference between the private interface and the public interface. Visibility modifiers exist in the first place so that you can do information hiding and clean polymorphism. If you have to allow friend access to so many different classes then your public interfaces are not clean. You have a bunch of classes depending on each other's implementation details in order to work properly. The information hiding afforded by classes and interfaces is a feature of C++ specifically designed to reduce the complexity inherant in large software projects -- friend is a way to work around that limitation *when necessary*. And there certainly are times when it is necessary, but overuse makes code more and more complex, intertwined, buggy and unmaintainable. Use friends very sparingly in C++ -- enumerators should be friends of collections, yes, but if documents are friends of views, you might have a problem on your hands. In C\#, JScript .NET and VB.NET there are no friendly classes. Rather, there is private, public and internal -- in C\#, private **really is private**, but any class **in the same assembly** can party on internal data. This gives you a lot of the benefits of friendly classes while still being able to restrict access to stuff that is truly private.  

</div>

</div>

