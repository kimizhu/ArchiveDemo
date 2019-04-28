# Three Umpires

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/9/2009 7:01:00 AM

-----

Three baseball umpires are having lunch together. The first umpire says "Well, a lot of them are balls, and a lot of them are strikes, but I always calls 'em as I sees 'em."

The second umpire says "Hmph. I calls 'em as they *are*."

The third umpire slowly looks at his two colleagues and declares "They ain't *nothin'* until I calls 'em."

-----

Those of you unfamiliar with the bizarre rules of baseball might need a brief primer. Suppose the pitcher throws a pitch and the batter swings and misses. [Such a failure to strike the ball is, bizarrely enough, called a "strike"](http://www.amazon.com/gp/recsradio/radio/B000002MSU/ref=pd_krex_dp_001_006/176-2812138-0508753?ie=UTF8&track=006&disc=001), and counts against the batter; three strikes and the batter is out. But what if the batter fails to swing at all? If the umpire decides that the pitch was "inside the [strike zone](http://en.wikipedia.org/wiki/Strike_zone)" then the pitch counts as a strike against the batter. If the pitch was outside the strike zone then the pitch counts as a "ball" (another bizarre and confusing name; obviously the object pitched is also called a ball). If the pitcher pitches four "balls" before the batter accumulates three "strikes" then the batter gets to "walk" to first base for free. (At least the walk is sensibly named.)

Formally, the strike zone is reasonably well-defined by the rules (though as the Wikipedia article linked to above indicates, there are some subtle points left out of the definition.) But the formal definition is actually irrelevant; the rules of baseball also state that *a strike is any pitch that the umpire says is a strike*. Umpires are given wide lattitude to declare what is a strike, and there are no appeals allowed.

And hence the fundamental disagreement between the three umpires. The first umpire believes that whether a pitch was in the strike zone or not is a *fact* about an objective reality, and that the call is a sometimes-imperfect subjective judgment about that reality. The second umpire seems to be basically agreeing with the objective, materialist stance of the first umpire, and simply bragging about having 100% accuracy in judgment. The third umpire's position is radically different from the first two: that the rules of baseball say that *regardless* of the objective reality of the path of the baseball, what *makes* a pitch into a ball or a strike *is the umpire's call*, no more, no less.

I think about the three umpires a lot. The C\# language has a clear and mostly unambiguous definition of "the strike zone"; the specification should in theory allow us to classify *any* finite set of finite strings of text into either "a legal C\# program" or "not a legal C\# program". As a *spec author*, I take the third umpire's position: what the spec says is the definition of what is legal, end of story. But as an all-too fallible *compiler writer*, I take the first umpire's position: the compiler calls 'em as it sees 'em. Sometimes an illegal program accidentally (or deliberately; we implement a small number of extensions to the formal C\# language) makes it through the compiler. And sometimes a legal program is incorrectly flagged as an error, or cannot be successfully compiled because it causes the compiler to run out of stack space or some other resource. (Also, though I believe that the compiler always in theory terminates, there are ways to build short C\# programs that take exponentially long to analyze, making the compiler a sometimes *impractical* tool for deciding correctness.) But we calls 'em as we sees 'em, and if we get it wrong, then that's a bug.

But as a practical matter for our customers, the compiler is more like the third umpire: the arbiter of correctness, with no appeal. And of course I haven't even begun to consider the runtime aspects of correctness\! Not only should the compiler decide what programs are legal, it should also generate correct code for every legal program. And again, the code generator plus the CLR's verifier and jitter act like our third umpire; the *de facto* arbiter of what "the right behaviour" actually is.

