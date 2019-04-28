# Original: When Are You Required To Set Objects To Nothing?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/21/2004 10:41:00 AM

-----

NOTE: I have significantly revised this article.  What you are reading now is the archive of the original posting.  The updated version is here <http://blogs.msdn.com/ericlippert/archive/2004/04/28/122259.aspx>   It's getting into sailing/kite flying season in Seattle -- that is, it's light out after work again -- and my home dev machine is presently busted (bad hard disk, grr), so I haven't had much time to work on SimpleScript.  I'll try to get the module system done over the weekend and start in on the name binders.  We'll see how it goes. 

Until then, a quick follow up on my [earlier entry](http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/30/53120.aspx") on the semantics of Nothing in VBScript.  I'm amazed that I haven't blogged about this before; for some reason I just didn't get around to it until now.  I see code like this all the time: 

Function FrobTheBlob()  
  Dim Frobber, Blob   
  Set Frobber = CreateObject("BitBucket.Frobnicator")  
  Set Blob = CreateObject("BitBucket.Blobnicator")  
  FrobTheBlob = Frobber.Frob(Blob)  
  Set Frobber = Nothing  
  Set Blob = Nothing  
End Function 

What's the deal with those last two assignments?  Based on the number of times I've seen code that looks just like this, LOTS of people out there are labouring under the incorrect belief that you have to set objects to Nothing when you're done with them. 

First off, let me be very clear: I'm going to criticize this programming practice, but **that does NOT mean that you should change existing, working code** that uses it\!  If it ain't broke, don't fix it. 

**The script engine will automatically clear those variables when they go out of scope**, so clearing them the statement before they go out of scope is pointless.  It's not *bad* -- clearly the code works, which is the important thing -- but it needlessly clutters up the program with meaning-free statements.  As I've ranted before, in a good program every statement has a meaning.  When I see code like this, the first thing I think is [cargo cult programmer](http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx"). Someone was told that the magic invocation that keeps the alligators away is to set objects to Nothing when you're done with them.  They do, and hey, it works\!  No alligators\!  (I'm sorry, Bert, I can't hear you; I've got a banana in my ear.) 

Where the heck did this thing come from?  I mean, you don't see people running around setting strings to "" or integers back to zero.  You never see it in JScript.  You only ever see this pattern with objects in VB and VBScript.  I have a couple of theories: 

1\)     Some previous version of VB required this.  That would be news to me, but I suppose it's possible.  Anyone out there know? 

2\)     Someone misunderstood how to take down a circular reference, and the mistake passed into folklore.  Suppose you find yourself in this unfortunate situation: 

Sub BlorbTheGlorb()  
  Dim Blorb, Glorb  
  Set Blorb = CreateObject("BitBucket.Blorb")  
  Set Glorb = CreateObject("BitBucket.Glorb")  
  Set Blorb.Glorber = Glorb  
  Set Glorb.Blorber = Blorb  
  '  
  ' Do more stuff here  
  ' 

and now when the procedure finishes up, those object references are going to leak because they are circular.  But you can't break the ref by cleaning up the **variables**, you have to clean up the **properties**.  Perhaps the myth started when someone misunderstood "you have to set the properties to Nothing" and took it to mean "variables" instead. 

3\)     Overapplication of the good programming practice "throw away expensive resources early".  Consider this routine: 

Sub FrobTheFile()  
  Dim Frobber  
  Set Frobber = CreateObject("BitBucket.Frobber")  
  Frobber.File = "c:\\blah.database"  ' locks the file for exclusive r/w access  
  '  
  ' Do stuff here  
  '  
  Set Frobber = Nothing ' final release on Frobber unlocks the file  
  '  
  ' Do more stuff here  
  '  
End Sub 

Here we've got a lock on a resource that someone else might want to acquire, so it's polite to throw it away as soon as you're done with it.  In this case it makes sense to explicitly clear the variable in order to release the object early, as we're not going to get to the end of the scope for a while.  This is a particularly good idea when you're talking about global variables, which do not go out of scope until the program ends. 

Another -- perhaps better -- design would be to also have a "close" method on the object that throws away resources if you need to do so explicitly.  This also has the nice result that the close method can take down circular references. 

4\)     Some widely read documentation has this programming practice in it, and people copy it without thinking about the meaning.  I see this all over the net.  Here's a [random example](http://platinum.intersystems.com/csp/docbook/DocBook.UI.Page.cls?KEY=gax_appparts "http://platinum.intersystems.com/csp/docbook/DocBook.UI.Page.cls?KEY=gax_appparts") that I got from Google: (lightly edited for clarity) 

You can save an instance of a persistent object using its **sys\_Save** method.  
Dim status As String  
patient.sys\_Save  
patient.sys\_Close  
Set patient = Nothing  
Note that you must call **sys\_Close** on an object when you are through using it. This closes the object on the server. In addition you should set patient to *Nothing* to close the object in Visual Basic. 

Notice that calling the close method is a "must" but setting the variable to Nothing is a "should".  Set your objects to Nothing: it's a [moral imperative](http://www.eric.lippert.com/moral.wav "http://www.eric.lippert.com/moral.wav")\!  **If you don't, the terrorists have already won.**  (One also wonders what the string declaration there is for.  It gets worse -- I've omitted the part of the documentation where they incorrectly state what the rules are for using parentheses.  This one page is a *mass* of mis-statements -- calling all of them out would take us very far off topic indeed.) 

I would imagine that there are lots of these in the MSDN documentation as well. 

But this is all conjecture.  Anyone out there know the origin of this myth?  Was there some good reason to do this in the past?

