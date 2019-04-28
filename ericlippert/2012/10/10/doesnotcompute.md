# Does Not Compute

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2012 6:27:00 AM

-----

One of the most basic ways to think about a computer program is that it is a device which takes in integers as inputs and spits out integers as outputs. The C\# compiler, for example, takes in source code strings, and those source code strings are essentially nothing more than enormous binary numbers. The output of the compiler is either diagnostic text, or strings of IL and metadata, which are also just enormous binary numbers.  Because the compiler is not perfect, in some rare cases it terminates abnormally with an internal error message. But those fatal error messages are also just big binary numbers. So let's take this as our basic model of a computer program: a computer program is a device that either (1) runs forever without producing output, or (2) computes a function that maps one integer to another.

So here's an interesting question: are there functions which cannot be computed, even in principle on a machine with arbitrarily much storage, by *any* C\# program (\*)?

We already know the answer to that question. [Last year I pointed out that the Halting Problem is not solvable by any computer program](http://blogs.msdn.com/b/ericlippert/archive/2011/02/24/never-say-never-part-two.aspx), because the assumption that it is solvable leads to a logical contradiction. But the Halting Problem is just a function on integers. Let's say that the input of our function H is a number which when written out in binary is a Unicode string that might contain a C\# program. The output is 1 if the program is an illegal C\# program, 2 if it is a legal C\# program which halts, and 3 if it is a legal C\# program which does not halt. If it were possible to write a program that reliably computes function H and always terminates then it would be possible to use it to solve the Halting Problem, which we've shown is impossible. Therefore H is not a computable function.

Let's explore this a bit further. The "Turing Machine" model of computing is that a computer is a machine that has three kinds of storage: first, there's a fixed amount of "internal" storage that describes the current state of the processor, second, there is arbitrarily much "external" storage in the form of paper tape, disk drives, or whatever, that can contain binary data, and third, there is some way of identifying the "current position" being manipulated in the external storage. The Turing Machine also has strict rules that describe how to change the internal state, the external state, and the current position. One of the internal states is the "start" state, and one of the internal states is the "halt" state; once the machine gets to the halting state, it stops. Otherwise, it runs forever.

Without loss of generality, let's suppose that our Turing Machine's external storage is arbitrarily many bits, either zero or one, and that the internal storage is some fixed number of bits, say n. This is pretty restrictive, but we haven't actually lost anything fundamental here. Real computers of course give the appearance of manipulating storage that consists of 32 bit integers or 64 bit doubles or whatever, but at some level inside the processor, it is manipulating individual bits. There is no difference in principle between a machine that manipulates one bit at a time and a machine that manipulates 64 bits at a time; the latter is simply more convenient.

So then how many rules do we need to come up with for our Turing machine? A Turing machine with n bits of internal state has 2<sup>n</sup> possible states, and there are two possibilities for the value at the "current position" in the external state. (\*\*) So that means that there are 2<sup>n+1</sup> state transition rules. Each transition rule will have to encode three things: (1) what are the n bits of the new internal state? (2) what value should the external state be changed to? and (3) how should we update the current position?

Again without loss of generality, we can update the current position by decreasing it by one, increasing it by one, or leaving it the same. In practice that is inconvenient, but in principle that is enough. So those are three possibilities. Thus, each state transition rule is one of 2 x 2<sup>n</sup> x 3 possibilities. There are 2<sup>n+1</sup> state transition rules. Therefore the total number of possible Turing Machines that have n bits of internal storage is 3 x 2<sup>n+1</sup> raised to the 2<sup>n+1</sup> power, which, yes, grows pretty quickly as n gets large, but which is clearly a finite number.

Each one of these n-bit Turing Machines essentially computes a function. You start it up with the external storage in a particular state and the machine either runs forever, or after some finite number of steps it halts. If it halts, then the output of the function is the value left behind in the external storage.

Again without loss of generality, let's consider the value computed by each one of those possible Turning machines when the external storage is initially all zeros. When given that starting configuration, each of those Turing machines either runs for some number of steps and then halts with the result, or it runs forever. Let's ignore the ones that run forever. Of the ones that are left, the ones that terminate, one of them must run the longest (\*\*\*). That is, one of those machines that halts must have the largest number of steps taken before entering the halting state.

We therefore can come up with a function S that goes from integers to integers. The function S takes in n, the number of bits in the Turing Machine internal state, and gives you back the largest number of steps any of the possible n-bit Turing Machines that halts takes to halt. That is, S takes in the number of bits of internal storage and gives you back the amount of time you have to wait for the **slowest** of the n-bit machines that actually terminates, when it is started with empty external storage.

**Is S a computable function?** Can we write a computer program that computes it?

Your intuition should be telling you "no", but do you see why?

.

.

.

.

.

.

.

.

[Because if S were computable then H would be computable too\!](https://www.youtube.com/watch?v=ieuBkWHfCuc) All we'd have to do to compute H is to make a computer program that compiles a given C\# program into a Turing Machine simulator that starts with an empty tape. We take the number of bits of state, n, of that Turing Machine, and compute S(n). Then we run the Turing Machine simulator and *if it takes more than S(n) steps then we know that it must have been one of the n-bit Turing machines that runs forever*. We'd then be able to reliably compute H in finite time. Since we already know that H is not reliably computable in finite time then we know that S must not be computable either.

The argument that I'm advancing here is known as the "Busy Beaver" argument because the n-bit Turing Machine that runs the longest is the "busiest beaver". I've tweaked the way that it is usually presented; rather than the **n-bit** Turing Machine that **runs the longest** before terminating, the "busiest beaver" is traditionally defined as the **k-state** Turing Machine that **produces the largest output**. The two characterizations are essentially equivalent though; neither version of the function is computable.

An interesting fact about the busy beaver function (either way you characterize it) is that the function grows *enormously fast*. It's easy to think of functions that grow quickly; even simple functions like n\! or 2<sup>n</sup> grow to astronomical levels for relatively small values of n, like 100. But our busiest beaver function S(n) grows faster than *any* computable function. That is, think of a function that grows quickly where you could write a program to compute its value in finite time; the busiest beaver function grows *faster* than your function, no matter how clever you are in coming up with a fast-growing function. Do you see why? You've got all the information you need here to work it out. (\*\*\*\*)

-----

(\*) Of course, there is nothing special about C\#; it is a general-purpose programming language. We'll take as given that if there is a function that cannot be computed in C\# then that function cannot be computed by any program in any programming language.

(\*\*) Of course, we don't need to state transitions from the halting state, but, whatever. We'll ignore that unimportant detail.

(\*\*\*) Of course, there could be a tie for longest, but that doesn't matter.

(\*\*\*\*) Of course, even if the busiest beaver function did not grow absurdly quickly, the fact that it clearly grows more than exponentially is evidence that our proposed technique for solving the Halting Problem would be impractical were it not impossible. Compiling a non-trivial C\# program to a Turing Machine simulator would undoubtedly produce a machine with more than, say, 100 bits of state. There are an enormous number of possible Turing Machines with 100 bits of internal state, and the one that runs the longest before it halts undoubtedly runs longer than the universe will last.

