# Murky Research

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/1/2010 8:55:00 AM

-----

No computers today, but some interesting - and important - math. (And, happy Canada Day, Canadians\!)

"[Car Talk](http://www.cartalk.com)" is a popular weekly phone-in program that has been on [National Public Radio](http://npr.org) for several decades now, in which Bostonian brothers Tom and Ray crack wise and diagnose car (and relationship) problems. On many programs they feature a "puzzler". Here's the puzzler from a couple of weeks ago, which I reprint in full:

Tim and Jethro were happy to have their jobs at the new self-serve gas station in town. And, since the Farmer's Almanac had predicted this to be the coldest winter since the last ice age, they were happy to be working indoors, while the customers pumped their own gas.

This station was so modern that it had a video camera for each of the pumps, and a TV monitor that would show the rear of everyone's vehicles as soon as they pulled up to the pumps.

When the boredom of their jobs finally set in, Tim and Jethro began playing a little game. The game involved trying to figure out which customers had pulled up to a pump with the fuel door on the wrong side-- that is, facing away from the pump.

Now, they couldn't see the cars pull in to the gas station. The video cameras were only aimed at the back of the vehicles. So, there was no time during which they could see the side of a vehicle where the fuel door was located. They could only see the vehicle after it was in position to refuel.

They had to make their bets before the driver shut off the key and exited the vehicle-- before he dope slapped himself for pulling in on the wrong side.

Jethro was correct 99 percent of the time. Tim was correct about 50 percent of the time, because he was just guessing.

What did Jethro know that enabled him to tell when a driver had pulled up to the pump with the fuel door facing the wrong way?

When I heard that puzzler, the answer was immediately obvious to me. Imagine my surprise when the answer that was immediately obvious to me was *not* the answer Ray gave this past Saturday\! The answer given by Ray was that in almost all cars, the tailpipe is on the opposite side of the car from the fuel door. Jethro knows that if a car pulls up with their tailpipe on the same side as the pump then they're on the wrong side. Jethro is 99% accurate because almost all cars you actually see on the road follow this pattern; very few cars have a tailpipe in the middle, have two tailpipes, have the fuel door in the middle, have the tailpipe on the same side as the door, or other anomalies.

Now, I'm not willing to go as far as Tommy sometimes does and call BOOOOOOOOOOOGUS\! on this one; I believe them that this is a 99% reliable heuristic for deducing the side the fuel door is on, and that if Jethro knew that, then Jethro could consistently defeat Tim. (I note that we are not given the parameters of how Tim is guessing, but we have reason to assume that Tim is guessing by some process akin to flipping a fair coin.) Clearly, when Jethro and Tim play this game, about half the games are going to be ties (because Tim is guessing right at random), but of the non-ties, Jethro is going to win most of the time. I'm not disputing that.

However, there is an extremely important aspect of this analysis which has been ignored. **The percentage of drivers who drive up to the wrong side we know from experience is low.** The only times I have driven up to the wrong side of the pump and had to back out is when in a borrowed or rented car (and many cars now have an arrow on the fuel gauge telling you where the fuel door is.) The vast majority of customers will pull up to the correct side. **This additional fact, not given in the statement of the puzzler but reasonably assumed, changes the analysis.**

Let's call a car which pulls up to the wrong side a "positive" (for reasons which will become apparent later) and a car which pulls up to the correct side a "negative". Jethro and Tim each make a prediction of whether a given car is going to be a positive or a negative. Let's call a car whose fuel door side can be correctly predicted by Jethro to be a "normal" car and one which cannot, an "unusual" car.   

Let's assume that the percentage of "positive" drivers is 1%. Notice that this is the *same* percentage of cars that Jethro's heuristic predicts incorrectly. I've made the assumption that these percentages are about the same deliberately but really all that matters is that they are both small. We'll do a bit of fiddling with the numbers later to see what happens when we change those around. But for now, just assume that it is a coincidence that the rate of boneheaded drivers is roughly the same rate as the number of cars that Jethro cannot correctly predict the side of the fuel door.

Given that reasonable assumption, Jethro does not need to have his fancy heuristic in the first place\! Suppose we replace Jethro with Bob, who **always bets negative, regardless of the position of the tailpipe**. This strategy is 99% accurate because 99% of the drivers are negatives\!

That was the solution that was immediately obvious to me: *deny the premise of the question*. You can defeat Tim's coin-flipping random strategy by simply observing that negatives vastly outnumber positives; it is reasonable to always bet on negative.

Suppose Bob and Tim play this game a million times  (it's a long winter in Boston) and Bob uses the "always negative" strategy, which, as we know, will be accurate 99% of the time solely on the basis of distribution of negatives vs positives. On average there will be 990000 negatives and 10000 positives. Bob will predict the negatives with 100% accuracy and Tim will predict them with 50% accuracy. So for those 990000 cases, Bob wins 495000 times, ties 495000 times, and loses never. Bob will predict the positives with 0% accuracy, Tim will predict them with 50% accuracy. So for those 10000 cases, Bob wins never, ties 5000 times, and loses 5000 times. Of the million games, Bob wins 495000 times, ties 500000 times, and loses 5000 times. As we'd naively expect, Bob wins 99% of the games which are not ties. 

Now suppose Jethro and Tim play this game a million times and Jethro uses his fancy tailpipe strategy, which is 99% accurate on the basis not of distribution of negatives, but on *ability to detect normals*. The analysis appears to be just the same as before: There will be 990000 normals and 10000 unusuals. Jethro will predict the normals with 100% accuracy, the unusuals with 0% accuracy, blah blah blah, and will win 495000 times, tie 500000 times and lose 5000 times, same as Bob.

This demonstrates my point that Jethro doesn't need his fancy-pants strategy to beat Tim. Bob and Jethro both beat Tim exactly the same number of times, on average, with their respective strategies. But we can look deeper at Jethro's strategy:

Of those million trials, 99% of them will be normals. Of those normals, 99% of them will be negatives. Working out the percentages, on average we should see:

980100 negative normals - Jethro predicts negative, correctly  
**9900 positive normals - Jethro predicts positive, correctly  
9900 negative unusuals - Jethro predicts positive, incorrectly  
**100 positive unusuals - Jethro predicts negative, incorrectly.

Holy smackers\! Jethro predicts positive 19800 times and is wrong 50% of the time, even with an overall 99% accurate heuristic\! **Considering only those cases where Jethro says positive, he is on average no more accurate than Tim, who is flipping a coin.**

Now suppose instead of yokels trying to predict whether a driver is boneheaded, we have three doctors each trying to predict whether you have Tappet's Disease. Suppose further that 99% of the population is negative for Tappet's Disease: they do not have it. Dr. Jethro has a test for Tappet's Disease which is 99% accurate. (People for whom the test works are "normals", and 99% of people are normals.) Dr. Bob doesn't even bother to diagnose you, he just says "you're negative" every time. Dr. Tim flips a coin.

Suppose you go to all three doctors and they all say "you're negative". Remember, Dr. Bob and Dr. Jethro are both accurate 99% of the time, but Dr. Tim is only accurate 50% of the time. Clearly you have learned absolutely nothing from Dr. Bob, who didn't even look at you; the fact that he is 99% accurate tells you nothing about *you*. Clearly you have learned absolutely nothing from Dr. Tim; he flipped a coin right in front of you. But of every 980200 patients where Dr. Jethro says "negative", he is correct 980100 times and incorrect 100 times. Dr. Jethro's 99% accurate test is actually 99.99% accurate **when he says negative**. 

Put another way: odds of Drs. Bob and Tim being wrong when they predict negative are both 1 in 100. Odds of Dr. Jethro being wrong if he predicts negative are 1 in 10000, a hundred times less likely. Dr. Jethro is taking advantage of the low incidence of positives *and* the accuracy of his test in a way that the other two are not. Dr. Jethro is way, way more reliable than either of the other two, **provided that your result is negative**.

But what if the result had been positive? (Obviously Dr. Bob would not say positive, so let's ignore him.) Of Dr. Jethro and Dr. Tim, which should you trust? If Dr. Tim says positive then you have a 99 in 100 chance that Dr. Tim is wrong. If Dr. Jethro says positive then you have a 50 in 100 chance that Dr. Jethro is wrong. Dr. Jethro is clearly still the winner here, but **it is deeply counterintuitive to people that a test which is overall accurate 99% of the time can have a 50% false positive rate.  ** 

And of course the more rare the disease is, obviously the more likely it is that a negative result is correct; most people don't have the disease, so a negative result is likely. But the flip side is that the more rare a disease is, the more likely it is that a positive is a false positive: an artefact of flaws in the test.

Imagine if only one in ten thousand drivers pulled up to the wrong side. Jethro's 99% accurate heuristic would now be *worse* than Bob's "always guess negative" strategy because Jethro gets so many false positives.

This is a serious problem in medicine\! The worst false outcome is a false negative - that is, the test says you do not have the condition when really you do. That's why Dr. Bob's strategy is completely unacceptable; all his false results are false negatives. But as we've seen, the mathematics of the situation means that given a Dr. Jethro with a reasonably accurate test for a condition with low incidence, false negatives are very rare.

But false positives cause unnecessary, potentially harmful or expensive treatment, not to mention unnecessary anxiety. The mathematics of the situation is that **false positives are an extremely high percentage of positives when the inaccuracy of the test and the rarity of the disease are close to each other.** Tests for rare conditions have to be *incredibly* accurate for the false positive rate to be low. The inaccuracy of the test has to be **orders of magnitude less** than the rarity of the condition.

This sort of probability analysis is based on [Bayes' Theorem](http://en.wikipedia.org/wiki/Bayes%27_theorem) and it has many fascinating implications beyond just this quick sketch. It has implications in law, in spam detection, it comes up all over the place.

If Tom and Ray have any comments on this critique of their puzzler, I'd be happy to hear them.

