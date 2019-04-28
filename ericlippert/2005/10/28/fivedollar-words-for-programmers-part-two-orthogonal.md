<div id="page">

# Five-Dollar Words for Programmers, Part Two: Orthogonal

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/28/2005 10:00:00 AM

-----

<div id="content">

In geometry, "orthogonal" basically means the same thing as "perpendicular", or "at right angles".  The walls are orthogonal to the floor. But algebraists extend the meaning of "orthogonal" beyond mere perpendicularity; to an algebraist, two aspects of a system are orthogonal if one can be varied without changing the value of the other.  Imagine for instance that you were trying to describe how to get from one point in an empty room to another. A perfectly valid way to do so would be to say how many steps to go north or south, and then how many steps to go northeast or southwest.  This hockey-stick navigation system is totally workable, but it feels weird because north and northeast are not orthogonal -- you can't change your position by moving northeast without also at the same time changing how far north you are.  With an orthogonal system -- say, the traditional north-south/east-west system -- you can specify how far north to go without worrying about taking the east-west movement into account at all. Nonorthogonal systems are hard to manipulate because it's hard to tweak isolated parts. Consider my fish tank for example. The pH, hardness, oxidation potential, dissolved oxygen content, salinity and conductivity of the water are very nonorthogonal; changing one tends to have an effect on the others, making it sometimes tricky to get the right balance. Even things like changing the light levels can change the bacteria and algae growth cycles causing chemical changes in the water. Computer people further extend this algebraic notion of orthogonality to their systems. Consider a case I just came across while studying the C\# expression binding code, for example. C\# warns about unreachable code: if (false) {  
label:   // targetted but not reachable  
  foo(); // warning\! unreachable\!  
  if (abc) goto label;  
} and also warns about untargetted labels: public void foo() {  
  bar();  
label:  // reachable but not targetted  
  blah();  
}  "Reachable" and "targetted" are orthogonal in one sense. You can have a reachable targetted label, a reachable untargetted label, an unreachable targetted label and an unreachable untargetted label. But they are nonorthogonal in another sense, as I discovered this week when I almost seriously broke the code generator. The way our code generator works, we must generate IL for an unreachable label *if it is targetted by a reachable goto*. The notions of reachable and targetted get conflated in this one corner case, meaning that I cannot have a code generator that skips generating all unreachable code. This unexpected nonorthogonality makes it difficult to write a compiler that is both correct and understandable, so I'm trying to figure out ways to either eliminate the nonorthogonality, or capture the problem in a very specific piece of highly localized code that I can comment the heck out of. Now hold on a minute, I hear you saying. Surely if the goto is reachable then the label will be too\! My challenge to you guys is this: how can you have an unreachable label targetted by a reachable goto?

</div>

</div>

