# Speeding Can Slow You Down

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/1/2003 5:35:00 PM

-----

I hope all you readers living in the United States had a restful and enjoyable Thanksgiving holiday. I sure did.

I've been meaning to talk a bit about some of the performance issues you run into when tuning massively multi-threaded applications, like the ASP engine.  I'd like to start off by saying that I am by no means an expert in this field, but I have picked up a thing or two from the real experts on the ASP perf team over the years.

One of the most intriguing things about tuning multi-threaded applications is that **making the code faster can slow it down.** How is that possible? It certainly seems counter-intuitive, doesn't it?

Let me give you an analogy. 

Suppose you have an unusual road system in your town.  You have a square grid of roads with stoplights at the intersections. But unlike the real world, these are *perfect* traffic lights -- they are only red if there actually is another car in the intersection. Unlike a normal road, each road goes only one way, and has at most one car on it at a time.  Once a car reaches the end of the road, it disappears and a new car may appear at the start. Furthermore, there is a small number of drivers -- typically one or two, but maybe eight or sixteen, but probably not one for every car. The drivers drive a car for a while, then stop it and run to the next car\!  The drivers are pretty smart -- if their car is stopped at a red stoplight then they'll run to a stopped car that is not at a red stoplight (if one exists) and drive it for a while. 

In our analogy each road represents a **thread** and each stoplight represents a **mutex**.  A mutex is a "mutually exclusive" section of code, also known as a "critical section".  Only one thread can be executing in that code at one time. The car represents the position of the **instruction counter** in this thread.  When the car reaches the end of the road, the task is finished -- the page is served. The drivers represent the **processors**, which give attention to one thread at a time and then context switch to another thread.  The time spent running from car to car is the time required to perform a thread context switch.

Now imagine that you want to maximize the throughput of this system -- the total number of cars that reach the end of their road per hour.  How do you do it?  There are some "obvious" ways to do so:

  - hire more drivers (use more processors)
  - eliminate some stoplights by building overpasses at intersections (eliminate critical sections)
  - buy faster cars (use faster processors)
  - make the roads shorter (write pages that require less code)

You'll note that each of these is either expensive or difficult.  Perf isn't easy\! 

Now, I said that these are "obvious" ways to solve this problem, and those scare quotes were intentional.  Imagine a complex grid of roads with lots of stoplights, a moderate number of cars on the road, and two drivers.  It is quite unlikely that cars will spend a lot of time stopped at stoplights -- mostly they'll just breeze right on through.  But what happens when you throw another six drivers into the mix? Sure, more cars are being driven at any one time, but that means that the likelihood of congestion at stoplights just went up. **Even though there are four times as many drivers, the additional time spent at stoplights means that the perf improvement is less than a factor of four**. We say that such systems are not *scalable*. 

Or consider a moderately congested freeway system with a whole lot of cars, drivers and intersections.  Now suppose that you keep the cars, drivers and intersections the same, but you shrink the whole system down to half its previous size. You make all the roads shorter, so that instead of having eight stoplights to the mile, now you've got twenty. Does total throughput get better or worse?  In a real traffic system, that would probably make things worse, and it can in web servers as well.  The cars spend all their time at intersections waiting for a driver to run over to them. **Making code faster sometimes makes performance worse because thread congestion and context switches could be the problem, not total path length.**

Similarly, making the cars faster often doesn't help. In the real world of Seattle traffic, upgrading from my 140-or-so horsepower Miata to a 300 HP BMW isn't going to get me home any faster.  Getting a faster processor and shorter program only helps if the "freeway" is clear of traffic. Otherwise, **you sit there in your souped-up ultimate driving machine going zero at stoplights like everyone else.**  Raw power does not scale in a world with lots of critical sections.

When perf tuning servers, use the performance monitor to keep a careful eye on not just pages served per second, but on thread switches per second, processor utilization and so on.  If you cannot saturate the processor and the number of thread switches is extremely high, then what is likely happening is that the drivers are spending way too much time running from car to car and not enough time actually driving. Clearly that's not good for perf.  Tracking down and eliminating critical sections is often the key to improving perf in these scenarios.

