# Immutability in C\# Part Eleven: A working double-ended queue

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/12/2008 6:40:00 PM

-----

At long last, let's get to that double-ended queue. Sorry for leaving you in suspense\!

Last time we came up with a non-solution -- as commenters immediately discovered, the given algorithm and data structure are deeply inefficient.  However, the basic *idea* is sound. There are a number of ways to solve this problem, but the way I'm going to talk about today is a much more efficient variation on the theme of our previous unsuccessful attempt. That is, have **cheap access to the endpoints**, and have some kind of more expensive "cold storage" in the middle.

In order to solve this problem I'm first going to introduce some helper classes which solve a much simpler problem. I want to implement a very limited kind of deque, namely one which can only store one, two, three or four items. A "dequelette", if you'll allow the silly name.

Note that there is no "empty" dequelette. Since this class does not actually fulfill the contract for a deque, I'm not going to have it implement IDeque\<T\>. Rather, I'll just have a base class that these four dequelette types derive from:

 

public class Deque\<T\> : IDeque\<T\>  
{  
  private abstract class Dequelette  
  {  
    public abstract int Size { get; }  
    public virtual bool Full { get { return false; } }  
    public abstract T PeekLeft();  
    public abstract T PeekRight();  
    public abstract Dequelette EnqueueLeft(T t);  
    public abstract Dequelette EnqueueRight(T t);  
    public abstract Dequelette DequeueLeft();  
    public abstract Dequelette DequeueRight();  
  }

The implementations of dequelette classes One, Two, Three and Four are obvious; rather than take up a lot of space here, I'll put them on a [separate page](http://blogs.msdn.com/ericlippert/pages/deque-cs.aspx) along with the rest of the implementation.

OK. What does this trivial little class hierarchy buy us? We have an immutable deque which can contain one, two, three or four elements, and that's all. Useful? Yes.

Here's my new recursive definition of a deque: a deque of T's is:

  - empty, or
  - a single element of type T, or
  - a left dequelette of T, followed by a middle **deque of dequelettes of T**, followed by a right dequelette of T.

That bit in boldface is the key. The cold storage in the middle is NOT another deque **of items**, it is a deque **of dequelettes of items**. (In particular in this implementation it will furthermore be a deque of dequelettes where every dequelette is of size three. Each item in the cold storage holds *three* items of the outer deque.)

This use of the type system may be a bit confusing. Let's draw it out. Without loss of generality, let's suppose that we're enqueuing 1, 2, 3, 4, ... onto the right hand side of a deque, starting with the empty deque. The structures we built up go like this:

 

Deque\<int\>.Empty  
Deque\<int\>.SingleDeque (1)  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Empty -- One(2) )  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Empty -- Two(2, 3) )  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Empty -- Three(2, 3, 4) )  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Empty -- Four(2, 3, 4, 5) )

Now at this point we are about to enqueue 6, but there is no more room on the cheap storage at the right endpoint. We need to make more room, so let's take three of those out of there, make a dequelette out of them, and stuff them into the right hand side of the cold storage. Notice that the thing in the middle is a single-element deque which contains a dequelette. It is *not* a three-element deque of integers:

 

Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Single (Three(2, 3, 4) ) -- Two(5, 6))  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Single (Three(2, 3, 4) ) -- Three(5, 6, 7))  
Deque\<int\>( One(1) -- Deque\<Deque\<int\>.Dequelette\>.Single (Three(2, 3, 4) ) -- Four(5, 6, 7, 8))

Again, as we are about to enqueue 9, we're out of space in the cheap seats. Move stuff into cold storage. That causes the cold storage to go from a single-element deque to a two-element deque. Note the type of the innermost middle empty element is a deque of dequelettes, where each dequelette contains three dequelettes, each of which contains three ints. So the innermost cold storage will be storing *nine items per node when eventually we start pushing stuff into it.* 

Deque\<int\>(  
    One(1)  
    Deque\<Deque\<int\>.Dequelette\>(  
        One(Three(2, 3, 4))   
        Deque\<Deque\<Deque\<int\>.Dequelette\>.Dequelette\>.Empty  
        One(Three(5, 6, 7)))  
    Two(8, 9))  
  

Our last example was inefficient. Is this efficient?

When enqueueing, most of the time the operation is cheap, just creating a new dequelette. Every third enqueue is expensive; it requires us to do two operations: first, to turn a four-dequelette into a two-dequelette, and second, to create a three dequelette and enqueue it into the cold storage. Every ninth enqueue causes the cold storage to itself have to do an expensive operation on its cold storage. And every twenty-seventh operation requires... and so on.  If you sum up that series, you find that the most *expensive possible operation* costs O(log n), but those are so few and far between that the *average cost of enqueueing each element* is O(1). Which is *awesome*.

The other objection that my psychic powers are picking up on is "holy goodness, what horrors must that be wrecking upon the type system?" Relax. The generic type system is plenty buff enough to handle this. Remember, it's a logarithmic thing. A deque with a *million* integers in it will have a Deque\<Deque\<Deque\<... \<int\>...\> no more than, say, *twenty* deep. The generic type system is plenty buff enough to handle a recursively defined generic type a mere twenty levels deep.

Also, remember, this is a generic type, not a C++ template. All the code for that crazy generic type is generated **once** by the jitter, and then the *same* code is shared for every reference constructed type where the type argument is a reference type -- which in this case, is all of them. That's the great thing about generic types; because they guarantee at compile time that everything in the implementation will work out at runtime, **the code can be reused**.

What about dequeueing? Same deal. When the cheap storage on the end points gets down to a single element dequelette, just dequeue a three-dequelette from the code storage and use that as the new cheap storage. Again, **at least** **two thirds of all operations are cheap** and the price of the expensive ones grows slowly, so the *amortized* cost of each dequeue is constant.

As a more advanced topic, which I may come back to at some point in the future, we can also implement cheap **concatenation** of two deques if we allow the cold storage to hold both twos and threes, instead of just threes. I have an implementation but the code for that gets really quite tedious, so I won't show it here. Using twos or threes makes it potentially slightly more expensive than just using threes because you go from a guarantee of two thirds of the operations being cheap to only half being cheap. But on average the amortized cost is still constant per operation whether it is twos or threes.

If you're still having trouble grasping the details of this data structure, try drawing out all the references on a whiteboard. You get a very odd tree-like structure. With most trees, the nodes "in the middle" are the "obvious" ones because they are near the root. In this tree structure (and it is a tree structure in the final analysis\!) its the *leaf notes near the edges* which are the most "shallow" nodes. The middle is where the tree gets "deeper". This is exactly what you want for a structure where the stuff on the edges needs to be cheap to access compared to the stuff in the middle.

Incidentally, this kind of tree is an example of a "finger tree". The "fingers" are the special references to the various special "edge" nodes.

We could keep on going and talking about more and more advanced analysis of immutable structures, but I think I'll stop here. This gives the flavour of this style of programming pretty well I think, so no need to labour the point further.

**Acknowledgements:**

One of the reasons I write about this stuff is because it forces me to learn about it. I learned a lot, and I hope you did too. For this series, I am indebted to Chris Okasaki's excellent book "Purely Functional Data Structures"; the thesis it was based upon is [here](http://www.cs.cmu.edu/~rwh/theses/okasaki.pdf).  Also to Ralf Hinze and Ross Paterson's [excellent paper on finger trees](http://www.soi.city.ac.uk/~ross/papers/FingerTree.pdf).

**Coming up next**: I'm not exactly sure. I have a pretty kick-ass implementation of some very fun algorithms which demonstrate the *practical* advantages of programming in an immutable style. Or perhaps I'll change gears a bit and talk about something else entirely. Either way, it'll be a fabulous adventure I'm sure.

