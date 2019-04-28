# Good Names

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/6/2009 12:30:00 PM

-----

Imagine a door with an unusual handle. The handle is five feet off the ground and rotates upwards to open the door. The door has no lock. Is this a good door design?

Sorry. That’s not an answerable question. The purpose of almost every door is to prevent *something* from going through it without preventing *everything* from going through it. Some doors exist only to mitigate heat loss while allowing anything else through. Some doors exist to keep everyone out except authorized personnel. The goodness or badness of a door design depends entirely on how well or poorly it performs the task of preventing undesired access and allowing desired access.

If I were describing a jail cell door or bank vault door, that would be an absurdly bad design. But for the teacher’s supply room in a kindergarden classroom, it’s pretty good. It doesn’t prevent access to any adults, but the kids are unlikely to be able to get through without adult supervision. **Purpose matters.**

Last time I talked a bit about what makes a book title better or worse. These ideas can be extended to naming just about anything, but it is important when considering the goodness or badness of a name to keep in mind the purpose of the name. Frequently when we name types and methods, it’s like my book example. The purpose of the code, like the purpose of the book, is to provide a benefit to the consumer. The purpose of the name is usually (\*) to make it easy for the potential consumer to **find** the thing, and then quickly and accurately **evaluate** whether it is likely to provide the desired benefit.

With those purposes in mind, here are some guidelines that I use when trying to come up with a public name for something:

  - Everything in my [**Bad Names**](http://blogs.msdn.com/ericlippert/archive/2007/06/12/bad-names.aspx) post.  
  - The name should describe what the thing *is* (classes) or *does* (methods) or is *used for* (interfaces), as opposed to *how* it achieves any of those things.  
  - Names should describe the **unchanging** aspects of the nature of the thing.  
      
    A small example of how I got this wrong recently: the style that I use for adding responses to comments in this blog is called “yellowbox”. If I ever decide to change it the look of the blog and want to make it blue, I’m going to look pretty silly having a style called “yellowbox” that draws a blue box. I should have called it “response”; it’s always going to be for responses.  
  - The name should use the **vocabulary of the consumer**, not the **jargon of the mechanisms** used in the implementation.  
  - The name should be as **unique** as possible, so that searching for the words in it rapidly narrows the field down.  
  - The name should be as **precise** as possible.  
      
    How many “HandleSomething” methods have you seen? Usually these do something a lot more specific than “handling”. It might be important to the consumer to have a better idea of what you’re doing in there.  
  - The name should not have any **non-standard abbreviations**; FindCustRec is unlikely to be found by searching for “customer” or “record”.

Those are just a few off the top of my head. What are some of the criteria you use to come up with good names for types and methods?

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) There are unusual cases where names are deliberately chosen to work *against* those typical goals. Sometimes code is very special-purpose, designed to be used very rarely and then only by subject experts. In those cases, you do not want the code to be accidentally found by people who are unlikely to need it, and unlikely to use it correctly. In those cases, it’s desirable for the name to be laden with the jargon of the experts who will be using it. Doing so sends the message to potential users “if you do not know what these words mean, you probably should not be using this code”. Again, purpose matters.

