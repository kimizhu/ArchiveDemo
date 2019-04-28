# SimpleScript Part Five: Named Items and Modules

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/22/2004 5:29:00 PM

-----

 

Named Items 

"Named items" are what we call the "top level" objects of the host provided object model.  WScript in WSH, window in Internet Explorer, Response in ASP, are all named items.  A host tells an initialized script engine about named items via the aptly named AddNamedItem method on the IActiveScript interface. 

HRESULT ScriptEngine::AddNamedItem(const WCHAR \* pszName, DWORD flags) 

A few things should immediately seem a little weird about this interface.  

The First Four Flags 

First off, what are the flags?  There are six.  The first four are pretty straightforward: 

If SCRIPTITEM\_ISVISIBLE is set then the name of the named item is added to the global namespace.  

Uh, OK… why would you ever NOT want this set?  Didn't I just say that named items were specifically for injecting named object model roots into the engine?  What good is a named item if you can't see its name? 

That brings us to the second flag; if SCRIPTITEM\_GLOBALMEMBERS is set then all (immediate) children of the named item are treated as though they are themselves top-level objects/methods.  That's how in Internet Explorer you can say 

window.alert("hello"); 

or 

alert("hello"); 

and they do the same thing.  IE tells the script engine that window is a visible named item with global members. 

Now it makes a little more sense why you might want to have an invisible named item.  What if you had an object with lots of methods and properties that you wanted available in the global namespace, but the object itself didn’t have a sensible name?  I can't think of any script host offhand that does that, but the capability is there if you need it. 

There's a second reason why you might want a named item to be invisible, but we'll get to that in a minute. 

The third flag is SCRIPTITEM\_ISSOURCE.  If that's set then we know that this object is an event source.  If the language supports implicit event binding and the host moves the engine into connected state, we're going to need to know which named items to hook up to which events.  It can be very expensive to do that hookup, so this provides a simple optimization.  If the host knows that a particular named item does not source events, it can choose to not mark the named item as a source, and we therefore never spend any time trying to hook up event sinks to it. 

The fourth flag is SCRIPTITEM\_ISPERSISTENT.  Recall that I said a while back that when the engine goes back to uninitialized state after being initialized, we throw away "some" named items, where "some" was to be defined later.  Now you know -- the engine remembers named items marked as persistent.  Information about those named items is not thrown away until the engine is closed.  Also, cloning an engine is basically making a copy of the uninitialized state of an engine, so persistent named items get cloned when their engine gets cloned.  As we'll see, this fact has implications for our implementation. 

Modules 

I'm sure you have a general idea of what I mean by a "module", though if you Google [define:module](http://www.google.com/search?sourceid=navclient&ie=UTF-8&oe=UTF-8&q=define%3Amodule "http://www.google.com/search?sourceid=navclient&ie=UTF-8&oe=UTF-8&q=define%3Amodule") you'll see that everyone has a slightly different definition.  Modules in the script engine sense are philosophically fairly straightforward.  I often want to have some way to say "this collection of functions can play with each other, but are isolated from this other collection of functions".  I want to be able to resolve name collisions by having two methods with the same name coexist in different modules.  

Well, I want that stuff in languages designed for "programming in the large".  When using script languages, more often than not you simply don't need to chunk stuff into modules.  But, bizarrely enough, the script engines support modules, and in a pretty goofy way at that.  Of course, any language implementor can implement module semantics however they want, but I see no reason to mess around with the de facto standards.  Here's how modules work in VBScript and JScript: 

There is a "global" module.  Procedures defined in the global module are callable from any module.  This is where the "built in" methods in VBScript and JScript go.

Every named item is associated with a unique module, with some exceptions:

  - Named items with global members are associated with the global module.
  - Named items marked with SCRIPTITEM\_NOCODE are not associated with any module

All visible named items are added to the global namespace, except for named items marked as SCRIPTITEM\_CODEONLY. Those are just names of modules and are associated with no object. 

Important distinction: though all visible named items are added to the global module's namespace, procedures in the named items' modules are not visible from the global module.

Let me try to make this a little more concrete.  Let's consider a hypothetical declarative language that supports embedded imperative script.  You might want to have something like this: 

\<application\>  
      \<script\>  
            function toggle ( x )  
               if (x == false) return true else return false  
      \</script\>  
      \<form name="fred"\>  
            \<checkbox name="chuck" checked="false" /\>  
            \<button name="bob"\>  
                  \<caption\>Hello world\!\</caption\>  
                  \<click\>  
                        fred.checked = toggle(fred.checked)  
                  \</click\>  
                  \<script\>  
                        function foo()   
                        // button-specific code here

And so on.  Suppose the host defined fred as a visible named item with global members, bob and chuck as named items each with their own module.  Then code in bob's event handlers can access anything in the global application namespace and any member of fred.  But if we now add script to chuck, chuck's module cannot see bob's code.  chuck is free to define its own function foo, which will not collide. 

What if bob wants to access code in chuck's module?  We'll see in a later entry how chuck can do that -- there are many subtle issues involved and we'll need more infrastructure built into SimpleScript before we can tackle it. 

Where's the Object? 

One more weird thing -- **where's the object**?  The method takes a name and some flags, but no object.  

Let me answer that question with another question: suppose a persistent named item's object is thread affinitized, and you clone the engine and initialize the clone on a different thread?  That thing can't use the object from the previous thread\!  

To solve this problem, we defer getting the actual object until we need it.  The engine calls the site back and asks for the object.  I haven't implemented that code yet; I'll talk more about that next time. 

The Implementation 

I've added a named item list object to SimpleScript in [nameditemlist.cpp](http://blogs.msdn.com/ericlippert/articles/116369.aspx "http://blogs.msdn.com/ericlippert/articles/116369.aspx") but I haven't quite figured out how I want to implement modules yet.  

As you can see, I've rolled my own hash table rather than using hash\_map.  There was quite a long series of [comments](http://blogs.msdn.com/ericlippert/archive/2004/04/15/114094.aspx#114135 "http://blogs.msdn.com/ericlippert/archive/2004/04/15/114094.aspx#114135") the other day about the pros and cons.  I chose to roll my own rather than using an off-the-shelf solution for several reasons: 

  - the implementation does not require a rocket-science hash table.  It's very unlikely that we're going to get a huge number of named items in a script engine.  Even very complex web pages seldom add more than a couple dozen named items.  (In IE, every button, form, etc, is a named item.)  Thus, I've implemented a fixed-number-of-buckets hashing-with-chaining table with a very simple string hashing algorithm. 

  - I really have no clue how hash\_map works, and I want to spend zero time debugging my way through acres of template code if there's a problem.

  - I realized that I was starting to succumb to [Object Happiness](/ericlippert/archive/2004/03/31/105329.aspx#106869 "http://weblogs.asp.net/ericlippert/archive/2004/03/31/105329.aspx#106869") as I was building a templatized hash table of my own.  I'm going to need two or three hash tables in this project all told.  And its not like templates actually make the generated code smaller or more efficient (though in C\#, generics can -- but that's another story.)  I'm going to keep it simple and avoid getting Happy.

  - hash\_map requires that new throw exceptions.  As I've [mentioned before](http://blogs.msdn.com/ericlippert/archive/2004/03/25/96373.aspx), that gives me the shakes.  I don't like mixing C++ exception handling into what is fundamentally a COM program.

  - I *think* hash\_map is threadsafe for the applications I'm going to use it in.  I'm not *sure*.  That makes me nervous.  I *am* sure that my own code is going to be a lot more amenable to analysis with respect to the goofy script engine threading contract.

  - I ended up spending WAAAY more time **investigating** the template code than it took me to write the hash table **from scratch**.  I'm doing this in my spare time here folks...  

All those things totally outweigh the negligible benefits of re-using an existing generic map, no matter how bulletproof and performant it is. 

That reminds me, I wanted to talk more about the script engine threading model.  My [earlier post](http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/18/53041.aspx") on the subject glossed over some important details. 

My team hit our Whidbey Beta One Zero Bug Bounce yesterday, so I'm off for a long weekend far, far away from computers.  I'll cut it short right now; next week I'll pick it up again and talk a bit more about the threading model, the implications of engine cloning, and I'll kick around some ideas about how to implement modules and code blocks.

