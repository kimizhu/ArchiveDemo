<div id="page">

# Hide and seek

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/14/2010 8:40:00 AM

-----

<div id="content">

<div class="mine">

Another [interesting question](http://stackoverflow.com/questions/2845276/should-a-protected-property-in-a-c-child-class-hide-access-to-a-public-property) from StackOverflow. That thing is a gold mine for blog topics. Consider the following:

<span style="font-size: small;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span></span><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-size: small;">B  
</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">int</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> X() { </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">return</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> 123; }  
}  
</span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">D</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> : </span></span></span><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-size: small;">B  
</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">new</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">protected</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">int</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> X() { </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">return</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> 456; }  
}  
</span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"><span style="color: #000000;"> </span></span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">E</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"><span style="color: #000000;"> : D</span></span></span><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-size: small;">  
</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"><span style="color: #000000;">{  
  </span></span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public int</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"><span style="color: #000000;"> Y() { </span></span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">return</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"><span style="color: #000000;"> X(); } // returns 456  
}</span></span></span></span>  
class </span></span></span></span><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-size: small;">P  
</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">static</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">void</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> Main()  
  {  
    </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">D</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> d = </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">new</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">D</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">();  
    </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Console</span></span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;">.WriteLine(d.X());  
  </span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;">}  
}</span></span></span>

There are two possible behaviours here. We could resolve X to be B.X and compile successfully, or resolve X to be D.X and give a "you can't access a protected method of D from inside class Program" error.

\[UPDATE: I've clarified this portion of the text to address questions from the comments. Thanks for the good questions.\]

We do the former.To compute the set of possible resolutions of name lookup,  the spec says"*the set consists of all accessible members named N in T, including inherited members*" but D.X is not accessible from outside of D; it's protected. So D.X is not in the accessible set.

The spec then says "*members that are hidden by other members are removed from the set*". Is B.X hidden by anything? It certainly appears to be hidden by D.X. Well, let's check. The spec says "*A declaration of a new member hides an inherited member only within the scope of the new member.*" The declaration of D.X is only hiding B.X within its scope: the body of D and the bodies of declarations of types derived from D. Since P is neither of those, D.X is not hiding B.X there, so B.X is visible, so that's the one we choose.

Inside E, D.X is accessible and hides B.X, so D.X is in the set and B.X is not.

What's the justification for this choice? Doesn't this conflict with our rule that methods in more derived classes are better than methods in base classes?

No, it doesn't. Remember, the rule about prefering derived to base methods is to mitigate the brittle base class problem. So is this rule\! Consider this brittle base class scenario:

FooCorp makes Foo.DLL:  
  
<span style="font-size: small;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Foo</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">  
{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">object </span></span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;">Blah() { ... }  
} </span></span></span>

BarCorp makes Bar.DLL:

<span style="font-size: small;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Bar</span></span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;"> : Foo  
{  
</span></span></span><span style="font-size: small;"><span style="font-family: Consolas; color: #008000;"><span style="font-family: Consolas; color: #008000;"><span style="font-family: Consolas; color: #008000;">  // stuff not having to do with Blah  
</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;">} </span></span></span>

 ABCCorp makes ABC.EXE:

<span style="font-size: small;"> <span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public </span></span></span></span></span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span></span><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af; font-size: x-small;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-size: small;">ABC  
</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">static</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">void</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> Main()  
  {  
    </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Console</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;">.WriteLine((</span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">new</span></span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas; font-size: x-small;"><span style="font-size: small;"> Bar()).Blah());   
  }  
}</span>  
</span></span>

Now BarCorp says "You know, in our internal code we can guarantee that Blah only ever returns string thanks to our knowledge of our derived implementation. Let's take advantage of that fact in our internal code."

<span style="font-size: small;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">public </span></span></span></span></span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">class</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Bar</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> : Foo  
{  
  </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">internal</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">new</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">string</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;"> Blah()  
  {  
    </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">object</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> r = </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">base</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">.Blah();  
    </span></span><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;"><span style="font-family: Consolas; color: #2b91af;">Debug</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;">.Assert(r </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">is</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">string</span></span></span></span><span style="font-size: small;"><span style="font-family: Consolas;"><span style="font-family: Consolas;">);  
    </span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">return</span></span></span><span style="font-family: Consolas;"><span style="font-family: Consolas;"> (</span></span><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;"><span style="font-family: Consolas; color: #0000ff;">string</span></span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;">)r;  
  </span></span></span><span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas;"><span style="font-size: small;">}  
} </span></span></span>

<span style="font-family: Consolas; font-size: x-small;"><span style="font-family: Consolas; font-size: x-small;"></span></span>ABCCorp picks up a new version of Bar.DLL which has a bunch of bug fixes that are blocking them. **Should their build break because they have a call to Blah, an internal method on Bar?** Of course not. That would be terrible. This change is a *private implementation detail that should be invisible outside of Bar.DLL*. The fact that hiding methods are ignored outside of their scopes means that they can be safely used for internal implementation details without breaking downstream users.

 

(Eric is in Oslo at NDC; this posting was prerecorded.)

 

 

 

</div>

</div>

</div>

