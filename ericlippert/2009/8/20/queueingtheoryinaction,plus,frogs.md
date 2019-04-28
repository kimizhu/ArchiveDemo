# Queueing Theory In Action, plus, frogs

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/20/2009 9:32:00 AM

-----

Well that was a lovely vacation. It got off to a poor start but then it improved dramatically.

Suppose you've got an "entrance" that is producing some largish number of "customers" on some schedule. You've got a bunch of "servers" who are handling the customer requests. Once a customer request is satisfied, the customer leaves through an "exit". What happens when there are more customers arriving in quick succession than there are available servers to serve them? This is the domain of **analytic queueing theory**; this theory is germane to a great many human and technical problems, from obtaining a double cheeseburger, onion rings and a large... orange... drink... to routing telephone calls over satellite networks.

(Some of you might be wondering what on earth the first two paragraphs have to do with each other. It'll come together eventually, I promise.)

An interesting example of two different queue servicing algorithms is exemplified by two popular fast food restaurant chains. At restaurant "M", if there are, say, four cashiers then there are four queues. Customers arrive, choose a queue and wait. At restaurant "W", there is one long serpentine queue; when a cashier becomes available, the person at the front of the line goes to that cashier.

The principle downside of the W system is that the single queue *looks* like it will take much longer than four short queues in the M system, which can be daunting. But by almost every relevant objective metric, by almost every relevant social factor, and in almost every common real-world business scenario the W system is preferable:

  - The W system **requires no customer to make a choice based on incomplete information**; the M system basically presents the customer with a roll of the dice. Which queue is fastest? It depends not just on the competence of the cashier, but on whether the transactions pending in a given queue are simple or complex.
  - Suppose you are in the M system and two queues -- yours, and the one beside you -- are being serviced at approximately the same rate on average. It is entirely possible and indeed, perhaps common, that even though both queues are in the long run moving at the same rate, that due to sudden starts and stops in both queues you will *perceive* that you are spending more time "being passed" by the people next to you than "passing" them. It is quite possible for people in *both* queues to have this perception simultaneously\! **Everyone feels like they've chosen the wrong queue**, even if there is no "wrong queue" on average. In the W system, there is only one queue, so it is automatically the right one.
  - The W system is **fair**; the customer who has waited the longest is always served next. The M system is not fair; customers who happen to be in a "fast" queue can get served before customers in a "slow" queue where they are waiting behind a complex ongoing transaction (or worse, behind a customer who has reached the front of the queue before deciding what to order.)
  - The W system has in theory higher possible **throughput**; the only time that a customer with a fast transaction pending at the front of the queue has to wait a long time is in the unlikely situation that *every* cashier happens to be busy with a complex transaction. If *any* cashier is at present serving fast transactions, then they're clearing the front of the queue quickly. In the M system, many fast transactions can get delayed by a *single* slow one. This drives average throughput way down.
  - The W system is far more **flexible**. New cashiers can be added dynamically when the queue gets too long and removed to perform less pressing tasks when it shortens. In the M system, when a new cashier comes online there can be a disorganized rush to form a new queue; customers are again asked to make a decision about whether to try the new queue or stay in the old one, and this produces new opportunities for perceived unfairness. But worse, when a cashier is *removed* in the M system, what happens to their queue?

It is the importance of this last question that was driven home to me on day one of my vacation. Let's call the airline "D". The D Air Lines baggage check in Seattle-Tacoma International Airport operates on the "M" queueing model. You check yourself in at one of the kiosks, print out your boarding passes, pay your fifteen dollars to check a bag, and then choose one of ten or so queues.

Now, D has at least *in theory* eliminated one problem with the M model; the last thing the checkin system tells you is which queue to stand in. I do not know how that system decides which queue is best, or whether any customer actually reads the message. It's a subtle thing; I personally was not expecting the system to give me this information, so I could see how someone could miss it entirely and just pick any old queue.

But anyways, we're standing in our assigned queue and its moving along. We don't particularly care how long its taking because our flight has been delayed, allegedly by *one* hour, due to an "unexpected maintenance issue". So we have plenty of time. (As it turns out, we in reality have over *three* hours of extra time. Which is fine. Airline mechanics reading this: please take all the time you need to ensure that the wings stay on.) And as we're waiting, I point out something to my wife: *the conveyor belt that the luggage is supposed to be going on is not moving*. Or rather, it's moving by about one bag width per minute -- jerkily starting up and then halting a second later. Luggage is of course arriving at the front of the queues far, far faster than that, so quite an accumulation of luggage is forming. Most of the "servers" aren't having to walk around, but a few people are walking behind the counter, and it's quite comical to see them trying to manoevre around the increasingly tall stacks of luggage piled beside the now-saturated conveyor.

Leah informs me that her friend C used to be a baggage handler for D Air Lines, but was recently laid off. "Oh, is the recession and slowdown in air travel making for fewer working hours available?" I ask. No, apparently C claims that there was *plenty* of work for them to do, but the precarious financial state of the airlines has led them to lay off staff and overwork the remaining handlers.

So anyway, we get to the front of the queue and I look up, trying to make eye contact with the D Air Lines representative directly in front of me. She is fixedly looking at the floor and says loudly in the vicinity of the representative serving the queue beside her "I have to leave now."  Which she does, continuing to look fixedly at the floor as she walks away. The other representatives, all busy serving other customers, do not react to this news. They may not have heard it. Or, they might have interpreted it as mere *information* -- as it was stated -- rather than as a *request* for someone to deal with the now-abandoned queue.

I have plenty of time to wait. So I do. I continue to try to make eye contact with the representatives serving the queues on either side of me, but they are either looking at the person they're serving, or looking with dismay at the growing towers of baggage now thoroughly engulfing the unmoving conveyor belt. I wonder whether it is difficult to miss someone less than two metres away staring at you with an expectant smile for minutes on end. Perhaps D Air Lines trains its staff to be good at that, I muse.

The minutes continue to pass. The queue behind me continues to grow. I reflect upon how this is a failure not just of customer service, but of application of basic results in queueing theory that you'd think an airline would be good at. I realize that I have a blog article in here somewhere, which makes me happy. I realize that I am now thinking about work while I'm on vacation, which makes me vexed. So on balance, it's a wash.

About five minutes into this, the traveller behind me politely taps me on the shoulder and asks "You're going to Michigan too, right? Am I in the right line?"

I turn to half face her, and half face the D Air Lines employee who has been so successfully ignoring me and the couple dozen people behind me. "*Ma'am*," I say, "*look at the larger picture. You are standing in a queue that is being served by no one to put your bags on a conveyor belt that is not moving. Most of the baggage handlers who would be taking your bags off the belt have been fired, and even if there were baggage handlers, the aircraft cannot fly, and probably is not even at this airport. Clearly you and I have not chosen the wrong **queue**; we have chosen the wrong **airline***."

Amazingly enough, that aforementioned representative *immediately* starts servicing our queue.

Though no indication whatsoever that there had been any sort of problem in the first place was forthcoming, to his credit he was polite, seemed reasonably competent at taking my bag, and added my bag to the tower. As we walked away I looked back and saw the fellow now servicing both queues; those queues were now each running at half speed, which I suppose is better than nothing, though I imagine that the people who had chosen either queue were less than thrilled. The airplane did eventually arrive and we did eventually retrieve the bag, so it all worked out in the end.

Now you know why most airlines use the "serpentine" W model rather than the M model. It prevents some of these sorts of problems from happening in the first place.

Things improved considerably after that. A sampling of stuff I saw on my vacation:

  - Jupiter in opposition
  - Ganymede
  - the Perseid meteor shower
  - trout
  - frogs
  - toads
  - tadpoles
  - turtles
  - turkey vultures
  - swallows
  - hummingbirds
  - mergansers
  - loons
  - great blue herons
  - some kind of predatory bird that was against the sun so I couldn't see it clearly but I suspect it was an osprey
  - chipmunks
  - bunny rabbits
  - silver birch
  - snapdragons
  - bright green dragonflies
  - sheep
  - roosters
  - fossilized clams
  - suspiciously damaged wooden kayak paddles: that is, evidence of beaver-shark activity. But how did they get into my kayak storage over the winter? *Are beaver-sharks amphibious?*
  - my family
  - old friends

The flight home -- where the baggage check was based on the W model -- was uneventful.

And so, back to more fabulous adventures in coding. I hope you enjoyed the pre-canned posted I prepared before my vacation.

Coming up next: one more addendum about iterator blocks.

