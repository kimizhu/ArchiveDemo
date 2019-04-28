# Attack of the Undead Cycle Detector

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/17/2004 10:20:00 PM

-----

My old friend Rob made some comments on yesterday's post which deserve to be called out: 

Consider a software application that is extensible with plug-ins. If some of the plug-ins are dependent on others, then a partial sort is required to determine the order in which they are loaded so that errors can be prevented. Further, since this application has an active plugin developer community, the list of plug-ins is always growing.  
  
The advantages of using a partial sort algorithm as described becomes clear: the dependency graph is much easier to maintain over time than a compiled list -- especially once the number of plugins reaches a large number (50-100). Finally, if someone only wanted to install a subset of plug-ins, they could query the dependency graph with that subset to realize the ordering of their subset.  
  
 

Indeed, there are many practical uses for a partial order sort.  Another common technique is used in project files for large software projects.  The executable depends on the resources and the object code.  The resources depend on the bitmap files.  The object code depends on the source code, depends on the headers, depends on the interface definitions… The dependency graphs can get very complex, and recompiling the whole thing can get very expensive.  A partial order sort is vital when writing a tool that determines "you've changed a bitmap, so we only need to recompile the resources and the executable, but not the interface definitions." 

Incidentally, the upcoming [MSBuild tool](http://msdn.microsoft.com/msdntv/episode.aspx?xml=episodes/en/20040122VSNETAK/manifest.xml "http://msdn.microsoft.com/msdntv/episode.aspx?xml=episodes/en/20040122VSNETAK/manifest.xml") that will ship with Whidbey is a good example of the kind of thing [I was talking about the other day](http://blogs.msdn.com/ericlippert/archive/2004/03/04/83981.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/04/83981.aspx") -- an XML based declarative language which calls through to customized imperative code. 

This example also highlights why you need to be careful of cycles, so I'm looking forward to seeing what you'll say about cycle detection. All I know is that it's a hard problem (if it wasn't, then Go would be a lot easier to write AI opponents for). 

Cycle detection is actually pretty easy -- it's a simple modification of the algorithm.  We already know that "bottom" nodes -- nodes with no living children -- are the ones you must do first, and our algorithm basically searches the tree for bottom nodes, killing them off as it goes.  But you'll notice in our earlier algorithm that I kill off a node *as soon as I start looking at its children*.  I can do that in the acyclic case because I know that since there are no cycles, we'll never encounter this node again while looking at its children.  It's going to be dead shortly, so might as well make it dead now. 

But if there *are* cycles that include the node we're currently examining, then we *will* hit this node again as we recursively look through all the children.  How can we detect that case?  We can't simply error out if we hit a dead node, because it might be a different dead node that we've already removed correctly.  In short, we need to know the difference between nodes that are dead *because we've already handled them* and all their children, and nodes that are dead *because we are in the process of handling them*.  The latter are cycles. 

What we'll do is instead of marking every node as "dead" or "alive", we'll have three states: dead, alive and "wandering the night in ghostly torment".  When we start looking at a node's children, we mark it as undead.  If, during the examination of the children, we hit an undead node, we know that we've hit a cycle.  We don't mark a node as dead until all its children are dead too.  (This is what happens when you let a dev who likes horror movies write the implementation: somehow you end up with zombies and dead teenagers...) 

var alive = 1;  
var undead = 2;  
var dead = 3; 

function topoSort(dependencies) {  
  var state = \[\];  
  var list = \[\];   
  for (var dependency in dependencies)  
    state\[dependency\] = alive;   
  for (var dependency in dependencies)  
    visit(dependencies, dependency, list, state);  
  return list;  
} 

function visit(dependencies, dependency, list, state) {   
  if (state\[dependency\] == dead)  
    return;   
  **if (state\[dependency\] == undead)  
**    **throw "Hey, you've got a cycle in dependency " + dependency;  
**  **state\[dependency\] = undead;  
**  for (var child in dependencies\[dependency\])  
    visit(dependencies, dependencies\[dependency\]\[child\], list, state);   
  **state\[dependency\] = dead;  
**  list.push(dependency);  
} 

As for go boards, the cycle detection problems are tricky, yes, but are the least of your worries when writing a go AI.  For those of you not familiar with go, the rules are simple to state.  To briefly do so: 

  - there are two players, black and white

  - black goes first

  - you take turns placing stones of your colour on the intersections of a square graph

  - groups of enemy stones that are entirely surrounded are captured

  - captured stones are worth one point

  - empty territory is worth one point per unit surrounded by you

  - you can pass your turn, and the game is over when there are two passes in a row.

  - playing a suicidal move -- where the stone you played would be entirely surrounded and hence captured -- is illegal 

Those are all straightforward.  The tricky rule is for what are called "ko" situations.  Suppose the board has a particular configuration, call it Alpha.  You play, putting the board into position Beta.  If the opponent has a move that EXACTLY restores the board to state Alpha, it is illegal for them to immediately play that move -- because then you've just gone into a cycle.  Strategic construction of ko situations is apparently an important part of go strategy, but I am but a novice at go, so I can't really comment further on the importance. 

Detecting cycles to prevent illegal play is simply a matter of storing the previous board state and comparing it to the board state after the proposed move.  So that's not a problem.  But another rule, one of the most obscure in Go, is that if there are three ko situations on the board, the game is over because the previous rule is insufficient to prevent longer cycles.  I think that situation is a draw.  That's similar to the chess rule that if the same position is ever repeated three times, it’s a draw.  Writing code to detect "there are three ko situations on the board" is tricky, but not impossible. 

The hard part of writing a go AI is not the cycle detection, but the strategy.  On a typical chess board, there are about sixty possible moves, most of which are immediately discardable as stupid.  You can usually get it down to, say, ten reasonable moves.  That means that to look one move ahead, you have to evaluate ten board positions.  Two moves, one hundred.  Three moves, one thousand…  the smaller you can prune the branching factor, the slower the number of board evaluations grows and therefore the deeper you can search.  

You can do a good job of pruning in chess because in chess, especially the way computers play it well, all moves tend to decrease the number of pieces on the board.  Both tactical and strategic moves tend to have immediate and obvious consequences -- I can take his queen, I can get a passed pawn, I can get a defended bishop on a long diagonal -- which make them easy to evaluate.  And the goal in chess is very well defined -- put the opposing king in checkmate and the game is over.  

None of that is true in go.  In go, the branching factor is often in the hundreds.  Pieces are added to and removed from the board constantly, changing the branching factor dynamically.  In go, there are situations where playing a piece in one corner of the board influences the outcome of a battle on the other side of the board fifty moves later, but playing that same piece one square over has a completely different result.  It's not unlike playing five separate games of chess where the pieces occasionally leak across from one board to another.  And in go, a major problem for the AI is simply knowing when to give up\!  You don't want to pass until you are sure that you can't make your situation any better.  That's hard to know in go. 

That's why there are computers that can beat every chess player in the world but one, but there are no go computers that can beat even moderately ranked humans.

