# Implementing Event Handling, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/9/2005 6:35:00 PM

-----

Back in February I posted a bit about [how script hosts such as Windows Script Host dynamically hook up event sources (objects) to event sinks (chunks of script that run when the event fires.)](http://blogs.msdn.com/ericlippert/archive/2005/02/15/373330.aspx) It has become apparent from some questions I've received recently that it would be helpful to have a more detailed explanation of how event handling works in COM, and how that applies to late-bound scripting languages.

We'll start in the early-bound world and then move into the scripting world next week. Suppose you've got an object Timer that has methods Start and Stop, and event Tick. The methods are straightforward. We define an interface that has these methods on it: interface ITimer : public IUnknown  
{  
public:  
  virtual void Start(int interval) = NULL;  
  virtual void Stop() = NULL;  
}  
When users want to call those methods, they just get a pointer to the IUnknown of the object, QueryInterface it for ITimer, and then call the appropriate method on the virtual function table. But the user doesn't just want to call the timer. Not much point in that\! They want the timer to inform some *other* piece of code whenever a tick event occurs. Somehow the sink - the listener function - is going to have to be called. This means that the timer code has to be *running* and it has to *call some arbitrary code* that it knows nothing about. How the timer code manages to run itself is a topic for another day; today we'll just worry about getting the call from the source to the sink. The sink is going to have to somehow register itself with the source, and they'll negotiate what interface the source will call on the sink when the event occurs. In this case the sink wants to know when a tick happens, so the sink should implement something like interface ITick : public IUnknown  
{  
public:  
  virtual void Tick(ITimer \* pSource) = NULL;  
}  
OK, so suppose we have a source that implements ITimer and a sink that implements ITick. Somehow they have to get hooked up.  (Error checking removed.) pSource-\>QueryInterface(IID\_IConnectionPointContainer, \&pContainer);  
pContainer-\>FindConnectionPoint(IID\_ITick, \&pConnection);  
pConnection-\>Advise(pSink, \&cookie);  
The sink says to the source "*do you contain connection points that can call me back on ITick?"* The source says "*sure, here's an object that you can register yourself with that will call you back*". The sink then says "*Super, here's a pointer back to me, and please give me a 'cookie' number that I can use to uniquely identify this relationship so that I can shut it down later*." The source now has a pointer to the sink, so when a tick event occurs, the source can call the sink. Next time we'll explore what happens in the scripting world where the sink only speaks IDispatch.

