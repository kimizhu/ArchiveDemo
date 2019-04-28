# Virtual Methods and Brittle Base Classes

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/7/2004 3:28:00 PM

-----

Hey, I'm back\! And in my new location.

That was the longest and least relaxing vacation I've ever taken. Fun, yes. Relaxing, no. And to top it off, my kitchen is not done yet. We're shooting for being able to run water tonight and actually use appliances by tomorrow night, but we'll see how it goes.

Well, enough chit-chat. I wanted to talk a little about the brittle-base-class problem, and how JScript .NET deals with it.

Virtual Methods and Brittle Base Classes

One of the challenges inherent in writing larger programs is managing change over time. Few large programs are written once and never updated. Usually new features are implemented for new versions. Class-based programming allows for clean, object-oriented design but there are still some pitfalls to be wary of. One of the more insidious object-oriented programming pitfalls is the **brittle base class problem**. Here's how it usually goes:

You develop a very useful class for version one of your project. You have a "gadget" which can be "confusticated":

// Project Juliet Version 1  
class Gadget {  
  public function confusticate() { // . . . 

Some coworkers working on another application at your company realize that their "widgets" are a special case of your gadgets, so they subclass. After all, code re-use is one of the benefits of object-oriented programming. These particular widgets need to be "garbled", so they add a new method:

// Project Romeo Version 1  
class Widget extends Gadget {  
  public function garble( ) { // . . . 

A widget is confusticated the same as a gadget, thus they do not override the confusticate method. They get your library, compile up their code, test everything out and it is all good.

Six months after version one of your project ships you are hard at work developing the next version. In this version you have decided that gadgets can be garbled too. Furthermore, you decide that any confusticated gadget needs to also be garbled, so you modify your sources:

// Project Juliet Version 2  
class Gadget {  
  public function garble() { // . . .  
  public function confusticate() {  
    this.garble();  
    // . . . 

You have no idea that your coworkers have extended your class and already added a method to garble a widget. Furthermore, **they do not necessarily know that you have changed the base class.** That could be a very small change amongst thousands of lines of code and many other changes. Now what happens when they call confusticate on one of their widgets? The base class's confusticate method is called, but because garble is virtual it then calls the derived class's garble method.

There are two ways that could be seriously wrong. First of all, you might have implemented the base class fully expecting that the *gadget* garbling method would be called upon confustication, but now the *widget* garbling is performed. Second, **the widget implementers have no idea that confusticating can cause garbling.** From their perspective the only code that can call garble is *their* code\! **As far as they know they are the only ones who have written a garbling method.**

"Base" classes are well-named -- the behaviour of the derived classes depends upon the base classes having rock-solid behaviour. If the base class implementations are brittle, then the derived classes will not be robust either. This is just one brittle base class scenario; there are many variations on this scenario.

One way to prevent the brittle base class problem is to use assembly manifests and config files to ensure that you always bind against the version you tested against. But in the spirit of providing the flexibility of multiple solutions, JScript .NET also affords some techniques to mitigate the rittle base class problem.

Trapping the Error: The Versionsafe Switch

The lesson here is that every time you change a base class you have to test not only the base class but every single derived class. There really is no getting around that fundamental fact but there is a tool which can help in this particular situation. If you run the JSC compiler with /versionsafe then the potential disaster described above will not be *averted* but it will at least be *brought to your attention*. Specifying this flag makes it illegal to make a virtual function by accident. In other words, it changes the default behavior from "make a function with the same signature as a base class function an overriding virtual function" to " make a function with the same signature as a base class function an error."

That means that when the people working on Project Romeo version 2 go to compile up the Widget class using the new Gadget they will *immediately* get an error. You have added a garble method which matches the signature in a base class and they must therefore explicitly say whether their matching method is a virtual (override) or a non-virtual (hide) method. Which solution is correct depends on the semantics of all the interacting methods; the point is to flag the potential problem automatically rather than using a default behavior which might be incorrect.

Preventing the Subclassing

The crux of the brittle base class problem is that the providers of the base and derived classes each have no idea what the other is doing. It is extremely annoying to have bugs crop up that seem to be in your code because someone else did a poor job of writing a subclass. It is possible and indeed highly desirable to simply not let anyone subclass your classes without a compelling reason.

The attribute final is used to indicate that a class may not be subclassed in JScript .NET:

final class Gadget  
{ /\* . . . \*/ } 

Now when your coworkers try this, they get an error:

class Widget extends Gadget // Error, Type Gadget may not be extended  
{ /\* . . . \*/ } 

It might be the case that you do want Gadget to be extendible but do not want a particular method to be overridden. To do this you can make individual functions final:

class Gadget {  
  final public function Garble() { // . . .  
} 

class Widget extends Gadget { // OK, Gadget is not final   
  override public function Garble() { // Error, Garble is final

Note that **it is legal to *hide* a final base class method**. It is only illegal to *override* a final base class method.

Another technique would be to use inheritance demands, but that's a topic for another day.

