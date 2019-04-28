# When Are You Required To Set Objects To Nothing?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/28/2004 11:37:00 AM

-----

A quick follow up on my [earlier entry](http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx") on the semantics of Nothing in VBScript. I see code like this all the time:

 

Function FrobTheBlob()  
  Dim Frobber, Blob  
  Set Frobber = CreateObject("BitBucket.Frobnicator")  
  Set Blob = CreateObject("BitBucket.Blobnicator")  
  FrobTheBlob = Frobber.Frob(Blob)  
  Set Frobber = Nothing  
  Set Blob = Nothing  
End Function

What's the deal with those last two assignments? Based on the number of times I've seen code that looks just like this, lots of people out there are labouring under the incorrect belief that you have to set objects to Nothing when you're done with them.

First off, let me be very clear: I'm going to criticize this programming practice, but **that does NOT mean that you should change existing, working code** that uses it\! If it ain't broke, don't fix it.

**The script engine will automatically clear those variables when they go out of scope**, so clearing them the statement before they go out of scope seems to be pointless. It's not *bad* -- clearly the code works, which is the important thing -- but it needlessly clutters up the program with meaning-free statements. As I've ranted before, in a good program every statement has a meaning.

When I see code like this, the first thing I think is [cargo cult programmer](http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx"). Someone was told that the magic invocation that keeps the alligators away is to put a banana in your ear and then set objects to Nothing when you're done with them. They do, and hey, it works\! No alligators\!

Where the heck did this thing come from?; I mean, you don't see people running around setting strings to "" or integers back to zero. You never see it in JScript. You only ever see this pattern with objects in VB and VBScript.

A few possible explanations immediately come to mind.

**Explanation \#1:** (Bogus) Perhaps some earlier version of VB required this. People would get into the habit out of necessity, and when it became no longer necessary, it's hard to break the habit. Many developers learn by reading old code, so those people would pick up on the old practice.

This explanation is bogus. To my knowledge there has never been any version of VB that *required* the user to explicitly deallocate all objects right before the variables holding them went out of scope. I'm aware that there are plenty of people on the Internet who will tell you that the reason they set their objects to Nothing is because the VB6 garbage collector is broken. I do not believe them. If you've got a repro that shows that the GC is broken, I'd love to see it.

**Explanation \#2:** (Bogus) Circular references are not cleaned up by the VB6 garbage collector. You've got to write code to clean them up, and typically that is done by setting properties to Nothing before the objects go out of scope.

Suppose you find yourself in this unfortunate situation:

 

Sub BlorbTheGlorb()  
  Dim Blorb, Glorb  
  Set Blorb = CreateObject("BitBucket.Blorb")  
  Set Glorb = CreateObject("BitBucket.Glorb")  
  Set Blorb.Glorber = Glorb  
  Set Glorb.Blorber = Blorb  
  '  
  ' Do more stuff here  
  '

and now when the procedure finishes up, those object references are going to leak because they are circular.

But you can't break the ref by cleaning up the **variables**, you have to clean up the **properties**. You have to say

 

Set Blorb.Glorber = Nothing  
Set Glorb.Blorber = Nothing

and not

 

Set Blorb = Nothing  
Set Glorb = Nothing

Perhaps the myth started when someone misunderstood "you have to set the properties to Nothing" and took it to mean "variables" instead. Then, as in my first explanation, the misinformation spread through copying code without fully understanding it.

I have a hard time believing this explanation either. Because they are error-prone, most people avoid circular references altogether. Could it really be that enough people ran into circular ref problems and they all solved the problem incorrectly to cause a critical mass? As Tommy says on Car Talk: Booooooooooooogus\!

**Explanation \#3:** It's a good idea to throw away expensive resources early. Perhaps people overgeneralized this rule? Consider this routine:  

Sub FrobTheFile()  
  Dim Frobber  
  Set Frobber = CreateObject("BitBucket.Frobber")  
  Frobber.File = "c:blah.database" ' locks the file for exclusive r/w access  
  '  
  ' Do stuff here  
  '  
  Set Frobber = Nothing ' final release on Frobber unlocks the file  
  '  
  ' Do more stuff here  
  '  
End Sub

Here we've got a lock on a resource that someone else might want to acquire, so it's polite to throw it away as soon as you're done with it. In this case it makes sense to explicitly clear the variable in order to release the object early, as we're not going to get to the end of its scope for a while. This is a particularly good idea when you're talking about global variables, which are not cleaned up until the program ends.

Another -- perhaps better -- design would be to also have a "close" method on the object that throws away resources if you need to do so explicitly.  This also has the nice result that the close method can take down circular references.

I can see how overapplication of this good design principle would lead to this programming practice. It's easier to remember “always set every object to Nothing when you are done with it“ than “always set expensive objects to Nothing when you are done with them if you are done with them well before they go out of scope“. The first is a hard-and-fast rule, the second has two judgment calls in it.

I'm still not convinced that this is the whole story though.

**Explanation \#4:** I originally thought when I started writing this entry that there was no difference between clearing variables yourself before they go out of scope, and letting the scope finalizer do it for you.  **There is a difference though, that I hadn't considered**. Consider our example before:

 

Sub BlorbTheGlorb()  
  Dim Blorb, Glorb  
  Set Blorb = CreateObject("BitBucket.Blorb")  
  Set Glorb = CreateObject("BitBucket.Glorb")

When the sub ends, are these the same?

 

  Set Blorb = Nothing  
  Set Glorb = Nothing  
End Sub

versus

 

  Set Glorb = Nothing  
  Set Blorb = Nothing  
End Sub

The garbage collector is going to pick one of them, and which one, we don't know. If these two objects have some complex interaction, and furthermore, **one of the objects has a bug whereby it must be shut down before the other**, then the scope finalizer might pick the wrong one\! 

(ASIDE: In C++, the order in which locals are destroyed is well defined, but it is still possible to make serious mistakes, particularly with the bane of my existence, smart pointers. See [Raymond's blog](http://weblogs.asp.net/oldnewthing/archive/2004/05/20/135841.aspx)for an example.)

The only way to work around the bug is to **explicitly clean up the objects in the right order before they go out of scope**.

And indeed, **there were widely-used ADO objects that had this kind of bug**.   Mystery solved.

I'm pretty much convinced that this is the origin of this programming practice.  Between ADO objects holding onto expensive recordsets (and therefore encouraging early clears), plus shutdown sequence bugs, lots of ADO code with this pattern got written.  Once enough code with a particular pattern gets written, it passes into folklore that this is what you're always supposed to do, even in situations that have absolutely nothing to do with the original bug.

I see this all over the place. Here's some sample documentation that I copied off the internet:

 

You can save an instance of a persistent object using its **sys\_Save** method. Note that you must call **sys\_Close** on an object when you are through using it. This closes the object on the server. In addition you should set patient to *Nothing* to close the object in Visual Basic.  
  
Dim status As String  
patient.sys\_Save  
patient.sys\_Close  
Set patient = Nothing

Notice that calling the close method is a "must" but setting the variable to Nothing is a "should". Set your locals to Nothing : it's a moral imperative\! **If you don't, the terrorists have already won.** (One also wonders what the string declaration there is for. It gets worse -- I've omitted the part of the documentation where they incorrectly state what the rules are for using parentheses. The page I got this from is a *mass* of mis-statements -- calling all of them out would take us very far off topic indeed.)

I would imagine that there are lots of these in the MSDN documentation as well.

What is truly strange to me though is how **tenacious** this coding practice is. OK, so some objects are buggy, and sometimes you can work around a bug by writing some code which would otherwise be unnecessary. Is the logical conclusion “always write the unnecessary code, just in case some bug happens in the future?”  [Some people](http://www.codecomments.com/ASP/message193000.html)call this “defensive coding”.  I call it “massive overgeneralization“. 

True story: I found a performance bug in the Whidbey CLR jitter the other day. There's a bizarre situation in which a particular mathematical calculation interacts with a bug in the jitter that causes the jitter to run really slowly on a particular method. It's screwing up our performance numbers quite badly.  If I change one of the constants in the calculation to a variable, the problem goes away, because we no longer hit the buggy code path in the jitter.

They'll fix the bug before we ship, but consider a hypothetical. Suppose we hadn't found the bug until after we'd shipped Whidbey. Suppose I needed to change my code so that in runs faster in the buggy Whidbey CLR. What's the right thing to do?

**Solution One:**  Change the constant to a variable in the affected method.  Put a comment as long as your arm in the code explaining to future maintenance programmers what the bug is, what versions of the framework causes the problem, how the workaround works, who implemented the workaround, and how to do regression testing should the underlying bug be fixed in future versions of the framework.  Realize that there might be similar problems elsewhere, and be on the lookout for performance anomalies.

**Solution Two**: Change all constants to variables. And from now on, program defensively; **never use constants again --** *because there might someday be a future version of the framework that has a similar bug*. Certainly don't put any comments in the code. Make sure that no maintenance programmers can possibly tell the necessary, by-design uses of variables from the unnecessary, pointless uses. Don't look for more problems; assume that your heuristic solution of never using constants again is sufficient to prevent not only *this* bug, but *future* bugs that don't even exist yet. Tell other people that “constants are slower than variables“, without any context.  And if anyone questions why that is, tell them that you've been programming longer than they have, so you know best.  Maybe throw in a little “Microsoft suxors, open source rulez\!” rhetoric while you're at it -- that stuff never gets old.

Perhaps I digress. I'd like to take this opportunity to recommend the first solution over the second.

This is analogous to what I was talking about the other day in my posting on [Comment Rot](http://blogs.msdn.com/ericlippert/archive/2004/05/04/125893.aspx). If you hide the important comments amongst hundreds of trivial comments, the program gets harder to understand.  Same thing here -- sometimes, it is necessary to write bizarre, seemingly pointless code in order to work around a bug in another object.  That's a clear case of the *purpose* of the code being impossible to deduce from the *syntax*, so call it out\!  Don't hide it amongst a thousand instances of identical really, truly pointless code.

